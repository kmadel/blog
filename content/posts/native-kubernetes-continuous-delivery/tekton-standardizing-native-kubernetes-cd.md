---
title: "Tekton Pipelines: Standardizing Native Kubernetes Continuous Delivery"
series: ["Native Kubernetes Continuous Delivery"]
tags: ["CDF","CI","CD","Tekton","Kubernetes", "Native K8s CD"]
date: 2019-03-15T21:00:15-04:00
draft: false
photo: "/photos/k8s-cd/pipe-brickwall.jpg"
photoCaption: "Random Wall with Pipe, Richmond, VA<br>Photograph by Kurt Madel ©2019"
exif: "SONY RX-100 ISO 125 28.0mm ƒ/1.8 1/500"
dataFile: pipe-brickwall
part: 2
---
Perhaps the most exciting project that was announced as one of the four initial [Continuous Delivery Foundation (CDF)](https://cd.foundation/) projects is [Tekton Pipelines](https://github.com/tektoncd/pipeline/blob/master/README.md), which in the vein of the Kubernetes ecosystem naming conventions is from the Ancient Greek word for carpenter. It is also the youngest of the [four initial CDF projects](https://cd.foundation/projects/). Surrounded by industry stalwarts with Jenkins on one side and Spinnaker the other - and then there is the upstart Jenkins X that is just over a year old, but seems much much older in *tech years* compared to Tekton. Tekton is exciting because it is a fresh start at providing an open framework for purley Native Kubernetes Continuous Delivery. In the second part of this [series on Native Kubernetes Continuous Delivery](/series/native-kubernetes-continuous-delivery/) I will explore where Tekton came from, what it is and why it is important.

>To see exactly what I mean by Native Kubernetes Continuous Delivery [check out the first part of this series](../native-k8s-cd).

## Where did Tekton come from?
Tekton Pipelines [lives on GitHub](https://github.com/tektoncd/pipeline) as do so many important open source projects. The [tektoncd GitHub Organization](https://github.com/tektoncd) didn't even exist a week before this post and its website at https://tekton.dev was still under construction at the time of this post - that's how new Tekton is. Before last week, Tekton lived in the [***build-pipeline*** repository](https://github.com/knative/build-pipeline) under the [Knative GitHub Organization](https://github.com/knative) and is built on the work of [Knative build](https://github.com/knative/build). It went from being called **build pipeline** to **Pipeline CRD** and now finally to **Tekton Pipelines** in a matter of a few months. Google announced [Knative at Google Next last July](https://cloudplatform.googleblog.com/2018/07/bringing-the-best-of-serverless-to-you.html) to include **Knative Build** - a rudimentary native Kubernetes framework initially created for building the container images to be deployed as Knative serverless applications. However, a number of different Kubernetes tools quickly latched onto Knative build as a more general framework for native Kubernetes CD - even though this wasn't the original intent. So Knative build-pipeline (now Tekton) was born to provide an open, full-featured and standardized native Kubernetes CD solution - where Knative Build was not. Tekton's heritage is still visible in the location of the first release, [v0.1.0](https://github.com/tektoncd/pipeline/releases/tag/v0.1.0) was actually released to https://storage.googleapis.com/knative-releases/build-pipeline/latest/release.yaml - I would expect future releases to be released under *tektoncd-releases* or something of the like.

## What is Tekton?
Tekton provides native Kubernetes resources for building and executing CD pipelines. But what does that mean exactly? 

Well first, Tekton provides [Kubernetes Custom Resource Definitions (CRD)](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) for modelling CD pipelines. This is where the open standardization of CD on Kubernetes begins in my opinion. The CRDs provided by Tekton include:

* [Task](https://github.com/tektoncd/pipeline/blob/master/config/300-task.yaml) - A Tekton pipeline will always include one or more [Tasks](https://github.com/tektoncd/pipeline/blob/master/docs/tasks.md) with a Task consisting of one or more [steps](https://github.com/tektoncd/pipeline/blob/master/docs/tasks.md#steps). Each step, which is an [Init Container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) (but will soon be just a plain vanilla K8s container spec as discussed below), is run sequentially with the source being to be built mounted into `/workspace`. If you don't specify a `command` for a `step` then the default `image` entrypoint will be used.
* [TaskRun](https://github.com/tektoncd/pipeline/blob/master/config/300-taskrun.yaml) - A TaskRun will be auto-created by a PipelineRun for each Task in a Pipeline, but a TaskRun can also be manually created and used if you want to run a Task outside of a Pipeline. 
* [Pipeline](https://github.com/tektoncd/pipeline/blob/master/config/300-pipeline.yaml) - A [Pipeline](https://github.com/tektoncd/pipeline/blob/master/docs/pipelines.md) is a collection of PipelineResources, [Parameters](https://github.com/tektoncd/pipeline/blob/master/docs/pipelines.md#parameters) and one or more Tasks.
* [PipelineRun](https://github.com/tektoncd/pipeline/blob/master/config/300-pipelinerun.yaml) - A [PipelineRun](https://github.com/tektoncd/pipeline/blob/master/docs/pipelineruns.md) is used to run a Pipeline and the creation of a PipelineRun will trigger the creation of a TaskRun for each Task in the Pipeline being run.
* [PipelineResource](https://github.com/tektoncd/pipeline/blob/master/config/300-resource.yaml) - [PipelineResources](https://github.com/tektoncd/pipeline/blob/master/docs/resources.md) provide input and output to a Pipeline's subsequents Tasks - with an input perhaps being a Git repository and an output being a container image built from that Git repository. Other resources include the [cluster resources](https://github.com/tektoncd/pipeline/blob/master/docs/resources.md#cluster-resource) - could be used to deploy to other clusters, and [storage resources](https://github.com/tektoncd/pipeline/blob/master/docs/resources.md#storage-resource).

Secondly, Tekton provides an *engine*, the [tekton-pipelines-controller](https://github.com/tektoncd/pipeline/blob/master/config/controller.yaml), to execute the modelled pipelines. Steps in a given Task will be executed sequentially and Tasks in a pipeline are also currently executed sequentially in the order they are declared in the Pipeline - although the ability to execute Tasks in parallel is coming soon - as discussed below.

## Why is Tekton important?
As part of the Continuous Delivery Foundation, Tekton is positioned to be the first open standard for defining native Kubernetes CD pipelines, and actually the first open standard for defining CD pipelines on any platform. Just as Kubernetes allows you to have container orchestration in a cloud agnostic way, Tekton will allow you to have CD-pipelines-as-code that aren't locked into any particular flavor of Kubernetes or Cloud provider, and can be adopted by other CD tools as a standardized engine for executing pipelines. Of course a Tekton Pipeline will only run on Kubernetes - but where else would you want to run your CD Pipelines?

## What's next?
Tekton is not yet *full-featured*, but it is showing a lot of promise already. Remember, this is only [v0.1.0](https://github.com/tektoncd/pipeline/releases/tag/v0.1.0). There are a number of important features on the roadmap for Tekton. Two important features targeted for v0.2.0 is to allow Tasks within a Pipeline to run in parallel and to allow execution of steps in regular pod containers instead of Init Containers. 

* [The Directed Acyclic Graph (DAG) execution capability that was merged on March 1st](https://github.com/tektoncd/pipeline/pull/473) provides the ability to execute tasks in parallel. DAG execution also allows Pipeline Tasks to be connected so that one may run before another and so that Pipeline execution cannot be caught in an infinite loop. Note that the DAG execution doesn't yet support the ability to set a maximum number of parallel Tasks - so you may want to watch out for that when you upgrade to v0.2.0 on clusters where you are sharing resources with other workloads. 
* One implementation detail that Tekton inherited from Knative build is the use of Init Containers as a way of executing steps sequentially. However, Init Containers don't provide native Kubernetes log persistence, the use of side-car containers, parallelization of steps or integrate with third party logging or metrics. By [creating a custom waiting entrypoint](https://github.com/tektoncd/pipeline/pull/564) that overrides and then executes each steps specified (or default) entrypoint, v0.2.0 of Tekton will allow using regular pod containers and provide the benefits mentioned above.

Other features being designed and on the [2019 roadmap](https://github.com/tektoncd/pipeline/blob/master/roadmap-2019.md) include:

* [Conditional execution](https://github.com/tektoncd/pipeline/issues/27): Run a task based on the value of a pipeline parameter, in response to the failure of another task, or other condition.
* [Notifications](https://github.com/tektoncd/pipeline/issues/49): Add built-in support for Slack, email and GitHub PR commenting with the ability to perform different actions based on the outcome of a Pipeline - like success vs failure.
* [Paused task execution](https://github.com/tektoncd/pipeline/issues/233): Allow a task to be configured to pause execution until a certain condition is met. This would enable features such as pausing for manual approval before a Task is executed.
* [Pipeline Task Restart](https://github.com/tektoncd/pipeline/issues/50): The ability to resume a Pipeline at a specific step or Task.
* [Retries](https://github.com/tektoncd/pipeline/issues/221): Allow specifying the number of times to retry when a Task fails.
* [Pipeline Extensibility](https://github.com/tektoncd/pipeline/issues/238): Allow custom resources, [custom Task implementations](https://github.com/tektoncd/pipeline/issues/215) and custom logic inside of a Task.
* [Triggers](https://github.com/tektoncd/pipeline/blob/master/roadmap-2019.md#triggering): The ability to kick-off a Tekton Pipeline based on an event such as a GitHub webhook. Currently a Tekton Pipeline (or Task) must be initiated manually by applying a PipelineRun (or a TaskRun).
* [Better Examples](https://github.com/tektoncd/pipeline/blob/master/roadmap-2019.md#community-library): Provide numerous in-depth examples on key CD use cases and a community where it is easy to share such examples.

Looking forward to v0.2.0 and beyond!
{{< load-photoswipe >}}
