---
title: "Jenkins X Goes Native"
series: ["Native Kubernetes Continuous Delivery"]
tags: ["CI","CD","Tekton","Prow","Jenkins X","Kubernetes","Native K8s CD"]
date: 2019-05-06T04:10:15-04:00
photo: "/photos/native-kubernetes-continuous-delivery/neptune-bologna.jpg"
photoCaption: "Fontana di Nettuno, Piazza del Nettuno, Bologna, Italy<br>Photograph by Kurt Madel ©2019"
exif: "SONY RX-100 ISO 125 310.4mm ƒ/1.8 1/1000"
part: 4
draft: false
---
In two of the previous posts of [this series](/series/native-kubernetes-continuous-delivery/) I wrote about two [Native Kubernetes Continuous Delivery (Native K8s CD)](../native-k8s-cd/) solutions - [Tekton](../tekton-standardizing-native-kubernetes-cd/) and [Prow](../prow/). In this post we will explore how Jenkins X uses both of these for its own CD, but more importantly how Jenkins X has seamlessly integrated both of these *Native K8s CD* platforms (among numerous others) into one easily consumable package making it incredibly easy for any CD practitioner to implement and execute best-of-breed *Native K8s CD*.


## Why is Native K8s CD important?
Before we get to how Jenkins X went native, let's review what *Native K8s CD* is and why I believe *Native K8s CD* is the only way to do CD if you really care about CD impacting your business and keeping up with your competitors.

### What is Native K8s CD?
- First, the CD process itself must run on and be orchestrated by K8s. This by itself does not constitute *Native K8s CD* - as it is [very easy to run Jenkins in a Java Virtual Machine (JVM) in a container in a pod on K8s](https://github.com/helm/charts/tree/master/stable/jenkins), but the CD process or platform must run as loosely coupled microservices packaged as lightweight containers to be truly *Native K8s CD*. Afterall, you can [put a monolithic app server like WebSphere in a container](https://hub.docker.com/r/ibmcom/websphere-traditional) - but should you (the Docker image for **WebSphere traditional** is ***just*** **1GB**)?
- Secondly, containers must be used to execute the steps of a CD pipeline. Once again, this by itself does not constitute *Native K8s CD* as *JVM Jenkins* has had the ability to use containers for CD pipeline steps [as far back as 2014](https://github.com/jenkinsci/docker-plugin/commit/50ec99db8571c6740412963461494dde97159203) - as do many other CD tools - like GitLab for example which is no more *Native K8s CD* than *JVM Jenkins*.
- Finally, to be *Native K8s CD* the tool must leverage native K8s resources, or more specifically, must be built on top of [Custom Resource Definitions (CRDs)](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/). This allows the tool to tap important native K8s capabilities, easily integrate with other native solutions, and enables the transparent use of native Kubernetes tools like `kubectl` to interact directly with the tool's custom objects or CRDs. 

*JVM Jenkins* running on K8s does not fit this definition of *Native K8s CD*. And actually, believe it or not, neither did Jenkins X - at least not until [Jenkins X adopted *NextGen* Pipelines with Tekton](https://jenkins-x.io/news/jenkins-x-next-gen-pipeline-engine/).

### Wasn't Jenkins X already Native K8s CD? 
Cloud native - perhaps - but **100% Native K8s CD** - **NO**. Jenkins X was not fully *Native K8s CD* before using [Tekton](https://tekton.dev/) as its Pipeline execution engine - as we will see below. Running legacy or traditional tools as containers on Kubernetes doesn't make them *Native K8s CD*. Sure, it may provide some advantages by running those tools on K8s - like better scalability and fault tolerance - but those legacy tools don't take advantage of all the *Native K8s CD* features and capabilities. 

### Why did Jenkins X go Native?
- **Standardization:** Kubernetes has become the defacto standard for container orchestration via declarative infrastructure, the same thing is happening for Kubernetes CD Pipelines with Tekton and Jenkins X. And if you aren't already using lightweight, highly scalable containers to execute your CD toolchain then you are falling behind your competitors.
- **Lead, Not Follow:** Jenkins X is proving to be a leader in the *Native K8s CD* space by adopting and contributing to other key K8s CD related projects that Jenkins X then leverages to be the best comprehensive CD platform on K8s. Jenkins X contributors have made many important contributions to [Prow](https://github.com/kubernetes/test-infra/pulls?q=is%3Apr+author%3Accojocar) and [Tekton](https://github.com/tektoncd/pipeline/pulls?q=is%3Apr+author%3Aabayer) - among other important K8s projects.
- **Performance:** Older CI/CD solutions are based on legacy architecture that isn't highly scalable, has incomplete APIs and CLI, and is difficult to manage and update. *Native K8s CD* is lighter, faster, stronger and Jenkins X makes it easy to use.

### So why wasn't Jenkins X completely *Native K8s CD* before now? 
Jenkins X ran on and is orchestrated by K8s. Jenkins X used containers to execute the steps of CD pipelines. Jenkins X has even leveraged [its very own CRDs](https://github.com/jenkins-x/jx/blob/master/pkg/kube/crds.go) to provide native integrations with K8s - but up until recently, it still relied on *JVM Jenkins*, a non-native K8s solution that is very difficult to effectively scale, as the execution engine of its CD pipelines. And even with the very cool ephemeral [Serverless Jenkins](https://jenkins-x.io/news/serverless-jenkins/) capabilities, it still had a very much non-Native K8s pipeline execution engine.

## Native K8s CD and Beyond

### Jenkins X CRDs
[By embracing CRDs from the very beginning](https://jenkins-x.io/architecture/custom-resources/), Jenkins X has been able to transparently integrate the important *Native K8s CD* projects of [Prow](https://jenkins-x.io/architecture/prow/) and now [Tekton](https://jenkins-x.io/architecture/jenkins-x-pipelines/). The `PipelineActivity` CRD is a great example of how Jenkins X has leveraged native K8s capabilities to provide an abstraction layer that eases the integration of Jenkins X components and other tools. For example:

- It doesn't matter if Jenkins X uses a static *JVM Jenkins* or Prow to consume a GitHub webhook - this is totally abstracted from the Jenkins X end user. (Actually it does matter if you don't want your CD to miss important webhooks - since [Prow is the better solution for listening for GitHub webhooks](../prow/#prow-is-a-github-webhook-listener).)
- It doesn't matter if Prow kicks off a job using an ephemeral *JVM Jenkins* via the [Jenkinsfile Runner](https://github.com/jenkinsci/jenkinsfile-runner) or Knative Build or Tekton Pipelines - again, this is totally abstracted from the Jenkins X end user.
- It doesn't matter if you interact with your Jenkins X job via the [JX CLI](https://jenkins-x.io/commands/jx_get_activities/) or the Jenkins X UI or even [Prow's `deck` component](https://jenkins-x.io/architecture/prow/#deck).

Of course there is a lot going on under the hood of Jenkins X to make all of this possible. But Jenkins X has abstracted it in such a way that allows Jenkins X to seamlessly integrate cutting-edge technologies in a way that is mostly transparent to the end user.

### Jenkins X Gets GitOps
Weaveworks coined the phrase and concept of [GitOps back in 2017](https://www.weave.works/blog/gitops-operations-by-pull-request). So, [what is GitOps](https://redmonk.com/jgovernor/2017/08/08/oh-hai-gitops-what-is-gitops/) and why is it important? In its most basic definition, GitOps is the declarative management of infrastructure delivery and software delivery as code in Git repositories. And GitOps is a very important part of what makes Jenkins X such a powerful software delivery platform - even though GitOps is not an exclusive *Native K8s CD* capability. And even though any CD tool on any platform can take advantage of [GitOps with declarative configuration](https://github.com/kypseli/cb-core-mm/blob/kube-workshop/config-as-code.yml), setting up GitOps outside of Jenkins X is not a trivial exercise - especially when compared to getting it OOTB with Jenkins X.

For purely native K8s workloads, Jenkins X uses GitOps to manage the deployment environments for your applications. From [PR previews](https://jenkins-x.io/developing/preview/) to staging to production, or [any other environment you want to define](https://jenkins-x.io/commands/jx_create_environment/) - Jenkins X manages all changes/deployments as code in a Git repository.

Jenkins X also provides the ability to [manage your Jenkins X installation with GitOps](https://jenkins-x.io/getting-started/manage-via-gitops/#using-gitops-to-manage-jenkins-x) allowing you to easily track all changes to your Jenkins X installation as Git commits.

### Jenkins X, On the Prow - ChatOps & More
[Jenkins X uses Prow for its own CD](https://github.com/jenkins-x/prow-config-tekton/blob/master/prow/config.yaml). And if you read [my post on Prow](../prow.md), you already know how awesome it is at automating CD for GitHub Orgs and repos. But Jenkins X didn't stop with just using Prow for its own CD, Jenkins X integrates Prow with [Quickstarts](https://jenkins-x.io/developing/create-quickstart/) - so you don't have to know anything at all about Prow to take advantage of its features.

One really cool Prow feature that you get OOTB with Jenkins X is ChatOps. Prow [seamlessly marries GitOps and ChatOps](../prow/#prow-chatops) making both features even better. Prow's ChatOps allows you to interact with Jenkins X via [simple `/foo` commands](https://prow.k8s.io/command-help) in GitHub issues and PR comments.

For another interesting perspective on ChatOps, check out [this blog post on ChatOps with Jenkins X by Viktor Farcic](https://technologyconversations.com/2019/04/24/implementing-chatops-with-jenkins-x/).

### Re-tooling with Tekton
Jenkins X supports a number of Pipeline execution engines. In the beginning it was just a static *JVM Jenkins* engine. But over time, Jenkins X has added several more execution engines. 

Next was [serverless Jenkins](https://medium.com/@jdrawlings/serverless-jenkins-with-jenkins-x-9134cbfe6870) - basically an ephemeral *JVM Jenkins* via the Jenkinsfile Runner, and this was made possible by integrating Prow. The integration of Prow also allowed for the easy integration of Knative Build pipelines.

Jenkins X has since moved on to embrace [Tekton Pipelines](https://github.com/tektoncd/pipeline), its sister project in the newly established [Continuous Delivery Foundation](https://cd.foundation/projects/). The two youngest [CDF projects](https://cd.foundation/projects/) are building a Native K8s CD solution together.

Tekton is an open source project that provides [100% K8s resources (CRDs) as a standard industry specification](https://github.com/tektoncd/pipeline/blob/master/docs/README.md) for end-to-end CD pipelines. And [Jenkins X has integrated Tekton](https://jenkins-x.io/news/jenkins-x-next-gen-pipeline-engine/) in what is quickly becoming the default way to execute Jenkins X CD pipelines - of course there will still be [a *JVM Jenkins* **engine** available for traditional/non-native pipelines in Jenkins X](https://github.com/jenkins-x-apps/jx-app-jenkins). But with Tekton ([backed by Prow](https://github.com/kubernetes/test-infra/pull/11888)), as the pipeline execution engine, Jenkins X is now a truly *Native K8s CD* platform.

### So why not just use Prow and Tekton? 
There are [too many Jenkins X features](https://jenkins-x.io/about/features/) to list each one and future posts will dive into some specifics of different features. But for today we will just leave it at a comparison of a [**Jenkins X `jenkins-x.yml` Pipeline file**](#jenkins-x-pipelines) versus the [**Tekton Pipeline CRDs that Jenkins X generates**](#tekton-crds-generated-by-jenkins-x). In a future post - and as the Tekton integration and Tekton based Pipelines becomes more stable, I will explore the Jenkins X Pipelines and Tekton integration in more detail.

## What's Next & Beyond *Native K8s CD*

### Use Jenkins X

> How do we move from Jenkins to some kind of different CI/CD tool that can better manage these cloud-native primitives simpler and more effectively? -- *https://thenewstack.io/the-best-ci-cd-tool-for-kubernetes-doesnt-exist/*

The answer is simple - Jenkins X! You should [start using Jenkins X today](https://jenkins-x.io/getting-started/) if you aren't already.

### Look Mom, No Server - Serverless Applications with Jenkins X
Serverless applications/functions are the next evolution in software development and deployment - a shift from microservices to nanoservices. But just as it hasn't been easy for many organizations to shift their software delivery to a microservice architecture, it is an even bigger challenge to move to a nanoservices architecture with serverless functions. Jenkins X makes it much easier. API Gateways, service mesh, [Progressive Delivery](https://redmonk.com/jgovernor/2018/08/06/towards-progressive-delivery/), etc - they are all very complicated and challenging for most organizations to adopt - Jenkins X provides the same GitOps, ChatOps, etc for serverless functions as it has always provided for microservice applications. [Jenkins X has OOTB support](https://jenkins-x.io/developing/serverless-apps/) for developing and deploying serverless functions/applications with [Knative Serving](https://github.com/knative/serving) and [Gloo](https://github.com/solo-io/gloo). And Jenkins X does it in a cloud agnostic way - avoiding vendor lock-in. Getting started with serverless applications with Jenkins X is as easy as `jx create addon gloo`.
<br>
<br>
In the next post of this series I will dive into nanoservices architecture and delivering serverless functions with Jenkins X. But if you don't wan to wait until then to learn more, I recommend that you read this [post on Progresive Delivery with Jenkins X, Istio, Prometheus and Flagger](https://blog.csanchez.org/2019/03/05/progressive-delivery-with-jenkins-x-automatic-canary-deployments/) by Carlos Sanchez.
<br>
<br>
<hr>
#### Jenkins X Pipelines
***`jenkins-x.yml`***
```yaml
buildPack: none
pipelineConfig:
  pipelines:
    release:
      pipeline:
        agent:
          image: alpine
        stages:
          - name: A Working Stage
            steps:
              - command: echo
                args:
                  - hello
                  - world
              - command: ls
          - name: Another stage
            steps:
              - command: echo
                args: ['again']
```

#### Tekton CRDs Generated by Jenkins X
***PipelineResource***
```yaml
items:
- apiVersion: tekton.dev/v1alpha1
  kind: PipelineResource
  metadata:
    creationTimestamp: null
    name: kypseli-helloworld-nodejs-maste
  spec:
    params:
    - name: revision
      value: v0.0.8
    - name: url
      value: https://github.com/kypseli/helloworld-nodejs
    type: git
  status: {}
metadata: {}
```

***Pipeline***
```yaml
apiVersion: tekton.dev/v1alpha1
kind: Pipeline
metadata:
  creationTimestamp: null
  name: kypseli-helloworld-nodejs-maste
  namespace: jx
spec:
  params:
  - default: 0.0.8
    description: the version number for this pipeline which is used as a tag on docker
      images and helm charts
    name: version
  - default: "1"
    description: the PipelineRun build number
    name: build_id
  resources:
  - name: kypseli-helloworld-nodejs-maste
    type: git
  tasks:
  - name: a-working-stage
    params:
    - name: version
      value: ${params.version}
    - name: build_id
      value: ${params.build_id}
    resources:
      inputs:
      - name: workspace
        resource: kypseli-helloworld-nodejs-maste
      outputs:
      - name: workspace
        resource: kypseli-helloworld-nodejs-maste
    taskRef:
      name: kypseli-helloworld-nodejs-maste-a-working-stage
  - name: another-stage
    params:
    - name: version
      value: ${params.version}
    - name: build_id
      value: ${params.build_id}
    resources:
      inputs:
      - from:
        - a-working-stage
        name: workspace
        resource: kypseli-helloworld-nodejs-maste
      outputs:
      - name: workspace
        resource: kypseli-helloworld-nodejs-maste
    runAfter:
    - a-working-stage
    taskRef:
      name: kypseli-helloworld-nodejs-maste-another-stage
status: {}
```

***Tasks***
```yaml
items:
- apiVersion: tekton.dev/v1alpha1
  kind: Task
  metadata:
    creationTimestamp: null
    labels:
      jenkins.io/task-stage-name: a-working-stage
    name: kypseli-helloworld-nodejs-maste-a-working-stage
    namespace: jx
  spec:
    inputs:
      params:
      - default: 0.0.8
        description: the version number for this pipeline which is used as a tag on
          docker images and helm charts
        name: version
      - default: "1"
        description: the PipelineRun build number
        name: build_id
      resources:
      - name: workspace
        targetPath: workspace
        type: git
    outputs:
      resources:
      - name: workspace
        targetPath: ""
        type: git
    steps:
    - args:
      - hello
      - world
      command:
      - echo
      env:
      - name: DOCKER_REGISTRY
        value: 10.47.252.4:5000
      - name: PIPELINE_KIND
        value: release
      - name: REPO_OWNER
        value: kypseli
      - name: REPO_NAME
        value: helloworld-nodejs
      - name: JOB_NAME
        value: kypseli/helloworld-nodejs/master
      - name: APP_NAME
        value: helloworld-nodejs
      - name: BRANCH_NAME
        value: master
      - name: JX_BATCH_MODE
        value: "true"
      - name: VERSION
        value: ${inputs.params.version}
      - name: BUILD_ID
        value: ${inputs.params.build_id}
      - name: PREVIEW_VERSION
        value: ${inputs.params.version}
      image: alpine
      name: step2
      resources: {}
      volumeMounts:
      - mountPath: /etc/podinfo
        name: podinfo
        readOnly: true
      workingDir: /workspace/workspace
    - command:
      - ls
      env:
      - name: DOCKER_REGISTRY
        value: 10.47.252.4:5000
      - name: PIPELINE_KIND
        value: release
      - name: REPO_OWNER
        value: kypseli
      - name: REPO_NAME
        value: helloworld-nodejs
      - name: JOB_NAME
        value: kypseli/helloworld-nodejs/master
      - name: APP_NAME
        value: helloworld-nodejs
      - name: BRANCH_NAME
        value: master
      - name: JX_BATCH_MODE
        value: "true"
      - name: VERSION
        value: ${inputs.params.version}
      - name: BUILD_ID
        value: ${inputs.params.build_id}
      - name: PREVIEW_VERSION
        value: ${inputs.params.version}
      image: alpine
      name: step3
      resources: {}
      volumeMounts:
      - mountPath: /etc/podinfo
        name: podinfo
        readOnly: true
      workingDir: /workspace/workspace
    volumes:
    - downwardAPI:
        items:
        - fieldRef:
            fieldPath: metadata.labels
          path: labels
      name: podinfo
- apiVersion: tekton.dev/v1alpha1
  kind: Task
  metadata:
    creationTimestamp: null
    labels:
      jenkins.io/task-stage-name: another-stage
    name: kypseli-helloworld-nodejs-maste-another-stage
    namespace: jx
  spec:
    inputs:
      params:
      - default: 0.0.8
        description: the version number for this pipeline which is used as a tag on
          docker images and helm charts
        name: version
      - default: "1"
        description: the PipelineRun build number
        name: build_id
      resources:
      - name: workspace
        targetPath: ""
        type: git
    outputs:
      resources:
      - name: workspace
        targetPath: ""
        type: git
    steps:
    - args:
      - again
      command:
      - echo
      env:
      - name: DOCKER_REGISTRY
        value: 10.47.252.4:5000
      - name: PIPELINE_KIND
        value: release
      - name: REPO_OWNER
        value: kypseli
      - name: REPO_NAME
        value: helloworld-nodejs
      - name: JOB_NAME
        value: kypseli/helloworld-nodejs/master
      - name: APP_NAME
        value: helloworld-nodejs
      - name: BRANCH_NAME
        value: master
      - name: JX_BATCH_MODE
        value: "true"
      - name: VERSION
        value: ${inputs.params.version}
      - name: BUILD_ID
        value: ${inputs.params.build_id}
      - name: PREVIEW_VERSION
        value: ${inputs.params.version}
      image: alpine
      name: step2
      resources: {}
      volumeMounts:
      - mountPath: /etc/podinfo
        name: podinfo
        readOnly: true
      workingDir: /workspace/workspace
    volumes:
    - downwardAPI:
        items:
        - fieldRef:
            fieldPath: metadata.labels
          path: labels
      name: podinfo
metadata: {}
```

***PipelineRun***
```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  creationTimestamp: null
  labels:
    branch: master
    owner: kypseli
    repo: helloworld-nodejs
  name: kypseli-helloworld-nodejs-maste
spec:
  Status: ""
  params:
  - name: version
    value: 0.0.8
  - name: build_id
    value: "1"
  pipelineRef:
    apiVersion: tekton.dev/v1alpha1
    name: kypseli-helloworld-nodejs-maste
  resources:
  - name: kypseli-helloworld-nodejs-maste
    resourceRef:
      apiVersion: tekton.dev/v1alpha1
      name: kypseli-helloworld-nodejs-maste
  serviceAccount: tekton-bot
  trigger:
    type: manual
status:
  conditions: null
```

{{< load-photoswipe >}}