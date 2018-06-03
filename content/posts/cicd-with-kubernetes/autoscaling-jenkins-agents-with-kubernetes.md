---
title: Autoscaling Jenkins Agents with Kubernetes
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","autoscaling"]
part: 2
date: 2018-05-27T07:09:15-04:00
draft: true
---
In [Part 1]({{< ref "segregating-jenkins-agents-on-kubernetes.md" >}}) of the series [CI/CD on Kubernetes](/series/cicd-on-kubernetes/) we used the `PodNodeSelector` to segregate the Jenkins workload - agents from masters - and explored a few Kubernetes admission controllers. In Part 2 of this CI/CD on Kubernetes series we will utilize the segregated `jenkins-agents` `node pool` as part of an autoscaling solution for the Jenkins agent workload, without impacting the availability or performance of the Jenkins masters `node pool`. But before diving into the autoscaling solution for Jenkins agents we will take a look at `ResourceQuotas` and `LimitRanges` - two more admission controllers.

> **NOTE:** The configuration examples used in this series are from a Kubernetes cluster deployed in AWS using [Kubernetes Operations or kops](https://github.com/kubernetes/kops) version [1.9](https://github.com/kubernetes/kops/blob/master/docs/releases/1.9-NOTES.md) with [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) enabled (which is the default authorization mode for kops 1.9). Therefore, certains steps and aspects of the configurations presented may be different for other Kuberenetes platforms.

## Resource Quotas and Autoscaling Don't Mix
Efficient and effective management of compute resources is an important part of successful CI/CD and it is no different with a Kubernetes cluster. The Kubernetes [`ResourceQuota`](https://kubernetes.io/docs/concepts/policy/resource-quotas/) object allows you to limit cluster resource utilization by `namespace` - ensuring that each workload or team in an organization has what they need without negatively impacting others. `ResourceQuotas`, for example, allow you to specify the total compute resources, such as cpu and memory (among many other resources), that may be used by a `namespace` - think of them as over-utilization guardrails for your CI/CD resources. However, it is a static configuration. So if you were to increase the number of `nodes` in a cluster you would have to manually update the `ResourceQuota` to make additional resources available for any `namespace` with a `ResourceQuota` assigned. However, the topic of this post is how to dynamically up-scale and down-scale cluster `nodes` for Jenkins agents - but it not currently possible to dynamically modify `ResourceQuotas`. There doesn't seem much point in having dynamic autoscaling while having a manual process to modify `ResourceQuotas` in order to actually utilize the autoscaled resources - so  `ResourceQuotas` and autoscaling by `namespace` don't mix. But you still may want to provide some guardrails within a particular `namespace` so that one individual Jenkins job does not adversely effect other jobs relying on the same pool of Jenkins agents. There must be some reasonable limit for each individual Jenkins agent `pod`. Also, for a Kubernetes cluster configured to autoscale `nodes` we still want to ensure that no single `pod` is configured to use more resources than are available on a single node as that `pod` will fail to launch. So we will define a `LimitRange` for the jenkins-agent `namespace` to assure realistic defaults and limits for each `container` in a `pod` and maximum cpu and memory for a `pod` overall. This will provide for more consistent and reasonable resource utilization and cluster autoscaling.

{{% codecaption caption="jenkinsAgentLimitRange.yml" %}}
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: jenkins-agent-limit-range
  namespace: jenkins-agents
spec:
  limits:
  - type: Pod
    max:
      cpu: 3
      memory: 12Gi
  - type: Container
    max:
      cpu: 2
      memory: 4Gi
    default:
      cpu: 2
      memory: 4Gi
    defaultRequest:
      cpu: 0.25
      memory: 500Mi
```
{{% /codecaption %}}

> **NOTE:** Even though we won't be using `ResourceQuotas` for this example - it is interesting that `ResourceQuotas` and `LimitRanges` work hand-in-hand. If a `ResourceQuota` is sprecified for cpu and/or memory on a `namespace` then `pod` `containers` must specify limits or requests for cpu/memory or the `pod` may not be created. So it is recommended that a `LimitRange` is created for the same `namespace` so default values are automatically applied to any `containers` that don't specify cpu/memory limits or requests.

## Why Autoscale
The [Jenkins Kubernetes Plugin](https://github.com/jenkinsci/kubernetes-plugin) provides the capability to provision dynamic and ephemeral Jenkins agents as `Pods` managed by Kubernetes. And while very useful, that is not the same thing as autoscaling the actual physical resources or `nodes` these `Pods` are running. Without true autoscaling there will be a static amount of resources available for your Jenkins agents workload (and any other applications utilizing those same Kubernetes nodes); and when and if all of those resources are consumed - any additional Jenkins workload will be queued. Furthermore, if you have consistent (or random) spikes in your CI/CD workload then autoscaling your agent capacity could be a solution for shorter queues - especially in a cloud environment like AWS, Azure, GCP, etc.

True autoscaling will allow your CI/CD to move as fast as it needs to - limiting slow-downs related to lack of infrastructure. But of course you will still want to set maximum limits to avoid large costs due to a mis-configured jobs or other user error - even with a `LimitRange` for the autoscaled `namespace`. And remember, down-scaling is just as important as up-scaling - it is the down-scaling that will help manage costs by removing under utilized `nodes`, while the up-scaling accelerates your CI/CD. Autoscaling will help reduce costs and maximize availability for your Jenkins agent workload.
## Kubernetes Autoscaler Project
The [Kubernetes Autoscaler](https://github.com/kubernetes/autoscaler) project provides a [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) component with support for AWS, GCP and Azure. A GA version was released alongside the release of Kubernetes 1.8.  The Cluster Autoscaler runs as a Kubernetes `Deployment` on your cluster and monitors the resource utilization of `node pools` or in kops terms [`InstanceGroups`](https://github.com/kubernetes/kops/blob/master/docs/instance_groups.md) - mapping to AutoScalingGroups in AWS. The Cluster Autoscaler will up-scale and down-scales `nodes` in a `node pool`/`instancegroup` based on `pod` consumption. Configuring and enabling the Kubernetes Cluster Autoscaler is very simple and straightforward. 

> **NOTE:** There are two other prominent types of autoscaling for Kubernetes: 1) The [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) and 2) The [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler). Put simply, the Horizontal Pod Autoscaler scales the number of `Pods` for a given application and is not a good fit for stateful, single `pod` applications like Jenkins masters or the ephemeral workload of Kubernetes based Jenkins agents. The Vertical Pod Autoscaler is of potential interest for both Jenkins masters and agents - as it will automatically adjust resource requirements for new and existing pods. One area of interest for the Vertical Pod Autoscaler is the capability to reduce the risk of containers running out of memory. This could be an excellent feature for Jenkins masters, allowing you to be more frugal with initial resource quotas with less risk. Definitely something worth taking a look at in a future post.

## Namespace Specific Autoscaling
We want to run a number of different workloads on our Kubernetes cluster - for example, in addition to providing dynamic Jenkins agents, we would also like to run Jenkins masters on the cluster and even deploy production applications to the same cluster. Of course you could set up separate clusters for each of these workloads, but the Kubernetes `Namespace` capability enables us to have true multi-tenant workload in the same cluster. For some workloads autoscaling is quite useful, whereas for other types of workloads autoscaling may not make sense, and may even be detrimental. Specifically, cluster autoscaling is detrimental for Jenkins masters. The Cluster Autoscaler will down-scale in addition to up-scaling - and when it scales down it will move/reschedule existing `pods` to other `nodes` in the pool so that `node` can be removed. This sudden unexpected stoppages is not a behavior we want for the Jenkins masters' workloads. But cluster autoscaling makes a lot of sense for Jenkins agents. So we will use the Jenkins agent `namespace` tied to a Jenkins agent `node pool` created in [part one of this series]({{< ref "segregating-jenkins-agents-on-kubernetes.md#putting-it-all-together" >}}) to ensure that only `nodes` used by Jenkins agents - and not other workloads - are autoscaled.

{{< figure src="k8s-cluster-autoscaling.png" title="Kubernetes Autoscaling Architecture Overview Diagram" >}}
### PodNodeSelector
The [PodNodeSelector](https://kubernetes.io/docs/admin/admission-controllers/#podnodeselector) admission controller that we [setup in part one of this series]({{< ref "segregating-jenkins-agents-on-kubernetes.md#configure-kops-admission-controllers" >}}) allows for targeted cluster autoscaling. To revisit, the `PodNodeSelector` allows you to place an annotation on a Kubernetes `namespace` that will automatically be assigned as a [`nodeSelctor`](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) label/value on all `pods` that are created in that `namespace`. We will configure the Cluster Autoscaler to specifically target the AWS AutoScalingGroup/`node pool`/`InstanceGroup` associated to the Jenkins agent `namespace` [created in part one of this series]({{< ref "segregating-jenkins-agents-on-kubernetes.md##jenkins-agent-instance-group" >}}) using the `PodNodeSelector`, so that only `nodes` used for Jenkins agent `pods` will be autoscaled. 

### Cluster Autoscaler Deployment
The following is an exerpt from the [cluster autoscaler example for running on a master node](https://github.com/kubernetes/autoscaler/blob/5ea4ddac977304302faa188f32b5965267374fc3/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-run-on-master.yaml) with the only modification being for the `nodes` argument for the `cluster-autoscaler` `app`:
{{% codecaption caption="clusterAutoscalerDeployment.yml" %}}
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    app: cluster-autoscaler
...
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --nodes=1:10:agentnodes.k8s.kurtmadel.com
...
```
{{% /codecaption %}}
The `cluster-autoscaler` deployment is in the `kube-system` `namespace` and is deployed to a master node.
