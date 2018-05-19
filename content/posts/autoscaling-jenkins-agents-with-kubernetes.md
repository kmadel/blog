---
title: Autoscaling Jenkins Agents with Kubernetes
date: 2018-05-19T12:49:15-04:00
draft: true
---
## Why Autoscale
The Jenkins Kubernetes Plugin provides the capability to provision dynamic and ephemeral Jenkins agents as Pods managed by Kubernetes. And while very useful, that is not the same thing as auto-scaling the actual physical (or virtual - VMs, EC2 instances, etc) resources for these Pods. So, without true auto-scaling, there will be a finite amount of resources available for your Jenkins agents (and any other applications utilizing those same Kubernetes nodes); and when and if all of those resources are consumed - any additional Jenkins workload will be queued. Furthermore, if you have have consistent (or even random) spikes in your CI/CD load then auto-scaling your agent capacity could be a solution - especially in a cloud environment like AWS, Azure, GCP, etc.

True auto-scaling will allow your CI/CD to move as fast as it needs to - limiting slow-downs related to lack of infrastructure within some maximum limits (of course it is important to set reasonable limits to avoid large costs due to a mis-configured jobs or other user error). And remember, scaling down is just as important as scaling up - it is the down-scaling that will save us cloud infrastructure costs, while it is the up-scaling that will save us CI/CD time - time to market - and hopefully, time to profit. 

In this post we explore setting up auto-scaling for Jenkins agents with Kubernetes running on AWS using kops. We will also explore some additional Kubernetes features to provide predictive auto-scaling; allow you to further avoid, or at least limit, the amount of time that any Jenkins job spends in the Jenkins build queue waiting for new infrastructure to come up. All the while only using the amount of infrastructure you need for any given window of time.

## Kubernetes Autoscaler Project
The Kubernetes Autoscaler provides a Cluster Autoscaler component with support for AWS, GCP and Azure. A GA version was released alongside the release of Kubernetes 1.8.  The Cluster Autoscaler runs as a Kubernetes Service on your cluster and monitors the resource utilization of an InstanceGroup. Configuring and enabling the Cluster Autoscaler is very simple and straightforward. However, as we will see, there are some additional complexities to be aware of in a scalable, highly-available multi-tenant Kubernetes cluster running Jenkins for multiple teams.

## The Kubernetes Admission Controllers
We could easily enable auto-scaling for our KOPS bases Kubernetes cluster on AWS without using any additional features besides what is provided by the Cluster Autoscaler. However, there are a few reasons we don’t want to do that.
We want to run a number of different workloads on our Kubernetes cluster - for example, in addition to providing dynamic Jenkins agents, we would also like to run Jenkins masters on the cluster and even deploy production applications to the same cluster. Of course you could set up separate clusters for each of these workloads, but the Kubernetes Namespace capability enables us to have true multi-tenant workload in the same cluster. We will create a 
We want to initiate up-scaling before it is actually needed so that our Jenkins jobs aren’t queued long or at all.
To achieve both these goals, we turn to Kubernetes Admission Controllers. Even if you don’t know what the Kubernetes Admission Controller is or does, chances are that you have benefited from one its many controllers. For example, if you use a default StorageClass or a ServiceAccount then you are depending on an Admission Controller.
PodNodeSelector
The PodNodeSelector Admission Controller allows you to place an annotation on a Namespace that will automatically be assigned as NodeSelctor label/value to all Pods that are created in that Namespace. So with KOPS on AWS, all we have to do is create an InstanceGroup and apply a Node Label.
Priority
The Priority Admission Controller allows you to add PriorityClass meta-data to a Pod on creation.
