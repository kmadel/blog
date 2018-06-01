---
title: Preemptive Autoscaling Jenkins Agents with Kubernetes
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","autoscaling","priority"]
part: 3
date: 2018-05-30T07:09:15-04:00
draft: true
---
In [Part 2]({{< ref "autoscaling-jenkins-agents-with-kubernetes.md" >}}) of the series [CI/CD on Kubernetes](/series/cicd-on-kubernetes/) we set up cluster autoscaling for the Jenkins agent `namespace`/`node pool`. In Part 3 of this CI/CD on Kubernetes series we will utilize yet another admission controller to enable preemptive autoscaling for the Jenkins agents `node pool`. We want to initiate **up-scaling** before it is actually needed to minimize queueing of Jenkins jobs. I like to refer to this as **preemptive autoscaling** because we will use [**priority preemption**](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/) to enable overprovisioning that will result in up-scaling before we need the new capacity for Jenkins agents. *Preemptive* autoscaling will help avoid, or at least limit, the amount of time that any Jenkins job spends in the Jenkins build queue waiting for new infrastructure to be scaled, but still only using the amount of infrastructure you need for any given window of time. 

However, there are some additional complexities to be aware of in creating a scalable, highly-available multi-tenant Kubernetes cluster running Jenkins and other workloads.

### Preemptive Autoscaling with Priority
Preemptive autoscaling depends on the [Priority](https://kubernetes.io/docs/admin/admission-controllers/#priority) admission controller. The **Priority** admission controller allows you to add `priorityClassName ` meta-data to a `pod` on creation and prioritize the scheduling of the `pods` in the cluster. When high priority `pods` are requested in a cluster with the Priority admission controller enabled and without enough `node` resources to accomodate them - the Kubernetes scheduler will evict lower priority `pods` to make room for pending, higher priority `pods`.  Coupling priority/preemption with the Cluster Autoscaler provides premeptive up-scaling - as lower priority `pods` are **preempted** to make room for higher priority `pods` the Cluster Autoscaler will initiate up-scaling to make room for the evicted low priority`pod(s)`. To enable *preemptive autoscaling* we will utilize *pause containers* ([read more about pause containers](https://www.ianlewis.org/en/almighty-pause-container))to create `pod(s)` with the purpose of consuming a certain amounut of cpu and/or memory resources to preemptively up-scale the `agent nodes` and reduce Jenkins job queue time. We will create a `deployment` of low priority `pods` using pause containers and with resource quotas and replicas configured to accomodate 4 average sized agents.

#### Priority Classes
We will create two `PriorityClasses` with one being utilized as a default for all `pods` that don't specify a `PriorityClass` and the other being used for the low priority `pods` with *pause containers*.

> **NOTE:** If you don't specify a `globalDefault` `PriorityClass` then any `pod` that does not specify a `priorityClassName` will be assigned a priority value of 0.


{{% codecaption caption="defaultPriorityClass.yml" %}}
```yaml
apiVersion: scheduling.k8s.io/v1alpha1
kind: PriorityClass
metadata:
  name: default-priority
value: 1
globalDefault: true
description: "Default PriorityClass for pods that don't specificy a PriorityClass."
```
{{% /codecaption %}}

#### Overprovisioning Deployment
{{% codecaption caption="overprovisioningDeployment.yml" %}}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: jenkins-agents
spec:
  replicas: 2
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources-1
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: "2"
            memory: "2Gi"
      - name: reserve-resources-2
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: "2"
            memory: "2Gi"
```
{{% /codecaption %}}