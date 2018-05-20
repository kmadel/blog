---
title: Autoscaling Jenkins Agents with Kubernetes
series: ["CICD for Kubernetes"]
part: 1
date: 2018-05-19T12:49:15-04:00
draft: true
---
## Why Autoscale
The [Jenkins Kubernetes Plugin](https://github.com/jenkinsci/kubernetes-plugin) provides the capability to provision dynamic and ephemeral Jenkins agents as Pods managed by Kubernetes. And while very useful, that is not the same thing as auto-scaling the actual physical (or virtual - VMs, EC2 instances, etc) resources for these Pods. So, without true auto-scaling, there will be a finite amount of resources available for your Jenkins agents (and any other applications utilizing those same Kubernetes nodes); and when and if all of those resources are consumed - any additional Jenkins workload will be queued. Furthermore, if you have consistent (or even random) spikes in your CI/CD load then auto-scaling your agent capacity could be a solution - especially in a cloud environment like AWS, Azure, GCP, etc.

True auto-scaling will allow your CI/CD to move as fast as it needs to - limiting slow-downs related to lack of infrastructure within some maximum limits (of course it is important to set reasonable limits to avoid large costs due to a mis-configured jobs or other user error). And remember, scaling down is just as important as scaling up - it is the down-scaling that will reduce costs, while it is the up-scaling that will save time - time to market - and hopefully, time to profit. 

In this post we will explore setting up auto-scaling for Jenkins agents with Kubernetes running on AWS using [Kubernetes Operations or kops](https://github.com/kubernetes/kops). While there are some aspects of the solution presented here that are unique to **kops**, the main ideas presented should be applicable to any cloud provider. We will also explore some additional Kubernetes features to provide predictive auto-scaling; allowing further avoidance, or at least limiting, the amount of time that any Jenkins job spends in the Jenkins build queue waiting for new infrastructure to be auto-scaled. All the while only using the amount of infrastructure you need for any given window of time.
## Kubernetes Autoscaler Project
The [Kubernetes Autoscaler](https://github.com/kubernetes/autoscaler) project provides a [Cluster Autoscaler component](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) with support for AWS, GCP and Azure. A GA version was released alongside the release of Kubernetes 1.8.  The Cluster Autoscaler runs as a Kubernetes Service on your cluster and monitors the resource utilization of node groups or in kops terms [`instancegroups`](https://github.com/kubernetes/kops/blob/master/docs/instance_groups.md). Configuring and enabling the Cluster Autoscaler service is very simple and straightforward. However, as we will see, there are some additional complexities to be aware of in creating a scalable, highly-available multi-tenant Kubernetes cluster running Jenkins for multiple teams.
## The Kubernetes Admission Controllers
We could quickly and easily enable auto-scaling for our kops based Kubernetes cluster on AWS without using any additional features besides what is provided by the Cluster Autoscaler. However, there are a few reasons we don’t want to do that.

1. We want to run a number of different workloads on our Kubernetes cluster - for example, in addition to providing dynamic Jenkins agents, we would also like to run Jenkins masters on the cluster and even deploy production applications to the same cluster. Of course you could set up separate clusters for each of these workloads, but the Kubernetes Namespace capability enables us to have true multi-tenant workload in the same cluster.
2. We want to initiate up-scaling before it is actually needed so that our Jenkins jobs aren’t queued long or at all. One might refer to this as predictive auto-scaling.

To achieve both these goals, we turn to [Kubernetes Admission Controllers](https://kubernetes.io/docs/admin/admission-controllers/#what-are-they). Even if you don’t know what the Kubernetes Admission Controller is or does, chances are that you have benefited from one its many controllers if you have interacted with a Kubernetes cluster. For example, if you use a default `StorageClass` or a `ServiceAccount` then you are depending on an Admission Controller.
##### PodNodeSelector
The [PodNodeSelector](https://kubernetes.io/docs/admin/admission-controllers/#podnodeselector) **Admission Controller** allows you to place an annotation on a Kubernetes `namespace` that will automatically be assigned as a [nodeSelctor](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) label/value to all `pods` that are created in that `namespace`. So with KOPS on AWS, all we have to do is create an InstanceGroup and apply a Node Label - in this case we will apply `jenkinsType: agent`. The we will create a dedicated Kubernetes `namespace` for agents with the `scheduler.alpha.kubernetes.io/node-selector` annotation:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  annotations:
      scheduler.alpha.kubernetes.io/node-selector: jenkinsType=agent
  name: jenkins-agents
``` 
Now all the `pods` that are created in that the **jenkins-agent** `namespace` will automatically have the `jenkinsType=agent` `node-selector` label/value applied to them and end up on the **agentnodes** `InstanceGroup`.
##### Priority
The [Priority](https://kubernetes.io/docs/admin/admission-controllers/#priority) Admission Controller allows you to add PriorityClass meta-data to a Pod on creation.
![Kubernetes Autoscaling](k8s-cluster-autoscaling.png)
