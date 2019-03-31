---
title: "Native Kubernetes Continuous Delivery: Why should you care?"
series: ["Native Kubernetes Continuous Delivery"]
tags: ["CI","CD","Tekton","Kubernetes","Native K8s CD"]
date: 2019-03-31T12:30:15-04:00
photo: "/photos/k8s-cd/pathfinder-of-the-seas.jpg"
photoCaption: "Pathfinder of the Sea, Monument Avenue, Richmond, VA<br>Photograph by Kurt Madel ©2019"
exif: "SONY RX-100 ISO 125 37.1mm ƒ/4.9 1/250"
part: 1
draft: true
---
Native Kubernetes Continuous Delivery (Native K8s CD) is, by definition, cloud native, so I wanted to start with the [CNCF definition of cloud native](https://github.com/cncf/toc/blob/master/DEFINITION.md):

>Cloud native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach.

>These techniques enable loosely coupled systems that are resilient, manageable, and observable. Combined with robust automation, they allow engineers to make high-impact changes frequently and predictably with minimal toil.

>The Cloud Native Computing Foundation seeks to drive adoption of this paradigm by fostering and sustaining an ecosystem of open source, vendor-neutral projects. We democratize state-of-the-art patterns to make these innovations accessible for everyone.

"Robust automation" is a key component of continuous delivery (and just to clarify, when I say continuous delivery that includes continuous integration). The definition also mentions "build" and "containers", and although those aren't mentioned in the same sentence, they come together quite nicely for continuous delivery. In this first part of this [series on Native Kubernetes Continuous Delivery](/series/native-kubernetes-continuous-delivery/) I will explain what Native K8s CD is and why it is important. But first let's take a brief look at the rise of containers, an integral component of *cloud native* and continuous delivery, before we explain exactly what I mean by Native K8s CD.

## Why?
I'll do my best [Steve Ballmer impression](https://www.youtube.com/watch?v=Vhh_GeBPOhs), but instead of "Developers, developers, developers" - *ad infinitum* - it is **containers, containers, containers!** And yes, I am sweating. 

Back in 2013, before Kubernetes was a thing, Docker was making Linux containers (LXC) much more accessible and Docker based containers took off ([and eventually dropped LXC](https://www.infoq.com/news/2014/03/docker_0_9)). At the same time, continuous integration (CI) was becoming a common best practice for software application delivery. The use of Docker containers with CI began to be adopted as an easy and efficient way to manage CI tools - compilers, testing tools, security scans, etc - to ensure reproducible and more manageable build and test environments. No more "but it worked on my machine!" OR slightly different versions of Java in every environment OR waiting for heavy-weight VMs to startup OR overly complex non-standard configuration to manage CI/CD infrastructure. Containers helped solve these CI/CD problems and many more.

Containers have been part of CI/CD for over half a decade now - and **if you aren't using containers as an integral part of your CI/CD then you are doing it wrong, period.** But there wasn't a manual on how to do it and managing all those containers can get pretty complicated. Sure, there were some popular CI and CD tools/platforms - but many of those tools and platforms were themselves trying to figure out exactly how to  most effectively leverage containers for CI/CD and it sometimes felt like trying to put a square peg in a round hole.

Fast forward to 2019 and not only has Kubernetes become the *de facto* standard for managing and orchestrating containers, Kubernetes is quickly becoming the best platform for continuous delivery. But why is Kubernetes being adopted as the best platform for CD?

Kubernetes is an excellent platform for executing CD because it has core features that naturally lend themselves to high-performing continuous delivery. These features include:

* **Scalability:** K8s is scalable (K8s handles scheduling) and applications that are architected for K8s are horizontally scalable. K8s allows you to easily scale your CD workload up and down.
* **Resilience:** K8s is fault tolerant. You can't really execute CD if your CD platform is down.
* **Built-in objects:** Resource, Config and Credential management is built-in. These objects form the core of any CD platform.
* **Extensibility:** K8s provides a number of extension points to include Custom Resource Definitions - ensuring that K8s is capable of providing a robust solution for any number of specialized use cases - like CD.

### So, what is Native Kubernetes Continuous Delivery?
But what exactly is Native Kubernetes Continuous Delivery and why should you care?

- First, the CD process itself must run on and be orchestrated by Kubernetes. This by itself does not constitute native K8s CD - as it is [very easy to run non-native CI/CD platforms on K8s](https://github.com/helm/charts/tree/master/stable/jenkins).
- Secondly, containers must be used to execute the steps of a CD pipeline. Once again, this by itself does not constitute native K8s CD as many CD platforms have been using containers for CD since 2014.
- Finally, to be native K8s CD the tool must leverage native K8s resources or more specifically, must be built on top of [Custom Resource Definitions (CRDs)](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/). This allows you to use native Kubernetes tools like `kubectl` to interact with the tool.

And why should you care? Should you want anything less from your CD platform than what you want for your deployed production applications? Your CD will touch every single one of your applications (at least it should) - so why wouldn't you have the most robust, fully automated, scalable and resilient CD platform possible? A Native K8s CD platform will give you all of that and more.

## How?
A number of large Kubernetes related projects have implemented their own CI/CD tooling and/or borrowed/built-on other such tooling from other Kubernetes projects. The best example of this is the [Kubernetes test-infra project](https://github.com/kubernetes/test-infra) - part of the [Kubernetes Testing Special Interest Group (sig-testing)](https://github.com/kubernetes/community/tree/master/sig-testing). The **test-infra** project epitomizes Native Kubernetes CI/CD and shows how CI/CD on Kubernetes has evolved over the last 4 years. The **test-infra** project provides "robust automation", and provides CI/CD for the entire Kubernetes project - sometimes over 10,000 CI jobs per day. The **test-infra** repository is home to a number of tools with the most important being [Prow](https://github.com/kubernetes/test-infra/tree/master/prow) - "a Kubernetes based CI/CD system." It is interesting that the test-infra project as a whole and even [Prow](https://github.com/kubernetes/test-infra/blob/master/prow/cmd/README.md#auxiliary-components), both had early support for Jenkins but have since moved to purely native K8s solutions for the execution of CI/CD jobs such as plain K8s pods, Knative Build and soon Tekton Pipelines.

[Knative Build](https://github.com/knative/build), part of the Knative project, was created solely to build and deploy Knative serverless applications. But a number of other projects began to leverage Knative Build for other CD purposes. And it quickly became evident that Knative Build was too low-level and lacked important CI/CD features. So ~~Knative Build Pipelines~~ ~~Knative Pipeline CRD~~ Tekton was created as a replacement.

[Jenkins X](https://jenkins-x.io/documentation/), a cloud native version of Jenkins, "is a CI/CD platform for Kubernetes." And Jenkins X - with its integration of Tekton and Prow - provides Native K8s CD.

### Why reinvent the wheel?
A really interesting thing about a number of these Native K8s CD projects is the synergy between them. Prow integrated Knative Build as a job execution engine - [adding Tekton and Jenkins X support as job execution engines is in the works](https://github.com/kubernetes/test-infra/pull/11888). Knative Build, Tekton and Jenkins X use Prow for their own CI/CD. It is also worth pointing out that [Tekton and Jenkins X are both inaugural projects](https://cd.foundation/projects/) of the new [Continuous Delivery Foundation (CDF)](https://cd.foundation/). I believe this synergy will continue, and with the growing momemtum of the CDF, will make it easier than ever for all CI/CD practitioners to adopt Native K8s CD.

In the next posts in this series I will explore some of these tools/platforms in greater detail, explore the synergy between them and explain how they provide Native K8s CD. I will be taking a look at: [Tekton](../tekton-standardizing-native-kubernetes-cd), Prow and then look at how Jenkins X brings everything together in one easy to use Native K8s CD platform.

{{< load-photoswipe >}}