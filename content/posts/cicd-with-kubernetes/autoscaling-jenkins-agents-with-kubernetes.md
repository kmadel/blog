---
title: Autoscaling Jenkins Agents with Kubernetes
series: ["CICD on Kubernetes"]
part: 2
date: 2018-05-21T06:29:15-04:00
draft: true
---
In [Part 1]({{< ref "segregating-jenkins-agents-on-kubernetes.md" >}}) of the series [CI/CD on Kubernetes](/series/cicd-on-kubernetes/) we explored Kubernetes admission controllers and used the `PodNodeSelector` to segregate our Jenkins workload - agents from masters. In Part 2 of the CI/CD on Kubernetes series we will utilize that segregation as part of an auto-scaling solution for the Jenkins agent workload, while leaving our Jenkins master(s) more static.
## Why Autoscale
The [Jenkins Kubernetes Plugin](https://github.com/jenkinsci/kubernetes-plugin) provides the capability to provision dynamic and ephemeral Jenkins agents as `Pods` managed by Kubernetes. And while very useful, that is not the same thing as auto-scaling the actual physical (or virtual - VMs, EC2 instances, etc) resources for these Pods. So, without true auto-scaling, there will be a finite amount of resources available for your Jenkins agents (and any other applications utilizing those same Kubernetes nodes); and when and if all of those resources are consumed - any additional Jenkins workload will be queued. Furthermore, if you have consistent (or even random) spikes in your CI/CD load then auto-scaling your agent capacity could be a solution - especially in a cloud environment like AWS, Azure, GCP, etc.

True auto-scaling will allow your CI/CD to move as fast as it needs to - limiting slow-downs related to lack of infrastructure within some maximum limits (of course it is important to set reasonable limits to avoid large costs due to a mis-configured jobs or other user error). And remember, scaling down is just as important as scaling up - it is the down-scaling that will reduce costs, while it is the up-scaling that will save time - time to market - and hopefully, time to profit. 

In this post we will explore setting up auto-scaling for Jenkins agents with Kubernetes running on AWS using [Kubernetes Operations or kops](https://github.com/kubernetes/kops). While there are some aspects of the solution presented here that are unique to **kops**, the main ideas presented should be applicable to any cloud provider. We will also explore some additional Kubernetes features to provide predictive auto-scaling; allowing further avoidance, or at least limiting, the amount of time that any Jenkins job spends in the Jenkins build queue waiting for new infrastructure to be auto-scaled. All the while only using the amount of infrastructure you need for any given window of time.
## Kubernetes Autoscaler Project
The [Kubernetes Autoscaler](https://github.com/kubernetes/autoscaler) project provides a [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) component with support for AWS, GCP and Azure. A GA version was released alongside the release of Kubernetes 1.8.  The Cluster Autoscaler runs as a Kubernetes Deployment on your cluster and monitors the resource utilization of node groups or in kops terms [`instancegroups`](https://github.com/kubernetes/kops/blob/master/docs/instance_groups.md) and map to AutoScalingGroups in AWS. Configuring and enabling the Kubernetes Cluster Autoscaler is very simple and straightforward. However, as we will see, there are some additional complexities to be aware of in creating a scalable, highly-available multi-tenant Kubernetes cluster running Jenkins for multiple teams.
## The Kubernetes Admission Controllers
We could quickly and easily enable auto-scaling for our kops based Kubernetes cluster on AWS without using any additional features besides what is provided by the default Cluster Autoscaler configuration. However, there are a few reasons we don’t want to do that.

1. We want to run a number of different workloads on our Kubernetes cluster - for example, in addition to providing dynamic Jenkins agents, we would also like to run Jenkins masters on the cluster and even deploy production applications to the same cluster. Of course you could set up separate clusters for each of these workloads, but the Kubernetes Namespace capability enables us to have true multi-tenant workload in the same cluster. We will use a specially configured Kubernetes `namespace` to create a dedicated `instancegroup` for the Jenkins agents.
2. We want to initiate **up-scaling** before it is actually needed so that our Jenkins jobs aren’t queued long or at all. One might refer to this as predictive auto-scaling. We will use [**priority preemption**](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/) to allow for overprovisioning that will result in scaling up before we need the new capacity of Jenkins agents. 

To achieve both these goals, we turn to [Kubernetes Admission Controllers](https://kubernetes.io/docs/admin/admission-controllers/#what-are-they). Even if you don’t know what the Kubernetes Admission Controller is or does, chances are that you have benefited from one its many controllers if you have interacted with a Kubernetes cluster. For example, if you use a default `StorageClass` or a `ServiceAccount` then you are depending on an Admission Controller.
{{< figure src="k8s-cluster-autoscaling.png" title="Kubernetes Autoscaling Architecture Overview" >}}
See the addendum below on [configuring non-standard Admission Controllers for kops]({{< relref "#configure-kops-admission-controllers" >}}).
##### PodNodeSelector
The [PodNodeSelector](https://kubernetes.io/docs/admin/admission-controllers/#podnodeselector) **Admission Controller** allows you to place an annotation on a Kubernetes `namespace` that will automatically be assigned as a [nodeSelctor](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) label/value to all `pods` that are created in that `namespace`. So with KOPS on AWS, all we have to do is create an InstanceGroup and apply a Node Label - in this case we will apply `jenkinsType: agent`. The we will create a dedicated Kubernetes `namespace` for agents with the `scheduler.alpha.kubernetes.io/node-selector` annotation:


Now all the `pods` that are created in the **jenkins-agent** `namespace` will automatically have the `jenkinsType=agent` `node-selector` label/value applied to them and end up on the **agentnodes** `InstanceGroup` defined below:

```yaml
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  creationTimestamp: 2018-05-01T11:28:57Z
  labels:
    kops.k8s.io/cluster: k8s.example.net
  name: agentnodes
spec:
  image: kope.io/k8s-1.8-debian-stretch-amd64-hvm-ebs-2018-02-08
  machineType: m5.large
  maxSize: 8
  minSize: 1
  nodeLabels:
    jenkinsType: agent
    kops.k8s.io/instancegroup: agentnodes
  role: Node
  subnets:
  - us-east-1b
```
Note the `nodeLabel` `jenkinsType` matched the `node-selector` in the **jenkins-agent** `namespace` above - both with a value of **agent**.
##### Priority
The [Priority](https://kubernetes.io/docs/admin/admission-controllers/#priority) Admission Controller allows you to add PriorityClass meta-data to a Pod on creation. This will allow the Kubernetes scheduler to evict lower priority Pods, making room for pending Pods with higher priorities.

## Configuring Non-Default Admission Controllers for KOPS {#configure-kops-admission-controllers}
For the version of kops used for this post - `1.9.3` - the following Admission Controllers are enabled by default (as you can see in the code for kops 1.9 [here](https://github.com/kubernetes/kops/blob/release-1.9/pkg/model/components/apiserver.go#L227) that is based on a [recommended set of admission controllers](https://kubernetes.io/docs/admin/admission-controllers/#is-there-a-recommended-set-of-admission-controllers-to-use):

```yaml
  kubeAPIServer:
    admissionControl:
    - Initializers
    - NamespaceLifecycle
    - LimitRanger
    - ServiceAccount
    - PersistentVolumeLabel
    - DefaultStorageClass
    - DefaultTolerationSeconds
    - MutatingAdmissionWebhook
    - ValidatingAdmissionWebhook
    - NodeRestriction
    - ResourceQuota
```
So we need to add the two additional admission controllers: `podNodeSelector` and `priority`. For **kops** you can do that by using the `kops edit cluster` command and if you are enabling additional admissoin controllers, such as these, on a new cluster you should do it before you apply the configuration or a rolling-update of all of your cluster nodes will be required. For the `podNodeSelector` it is as easy as adding it to the list of the `admissionControl` list for the `kubeAPIServer` configuration. 