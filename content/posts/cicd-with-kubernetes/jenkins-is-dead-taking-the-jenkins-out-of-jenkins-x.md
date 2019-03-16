---
title: Jenkins is Dead - Taking the Jenkins Out of Jenkins X
series: ["CICD on Kubernetes"]
tags: ["Jenkins","Jenkins X","CI","CD","Serverless","Tekton","Kubernetes"]
date: 2020-03-12T17:36:15-04:00
draft: true
---
The importance of Continuous Delivery (CD) has accelerated rapidly over the last several years. The recent launch of the [Continuous Delivery Foundation](https://cd.foundation/projects/) is definitely a sign of the continuing growth and significance of CD across enterprise IT and cloud computing. Not only has Jenkins been a very important part of this growth, it is one of the [initial projects of the Continuous Delivery Foundation](https://jenkins.io/blog/2019/03/12/cdf-launch/). 

## So why then is Jenkins dead?

Ok, so Jenkins isn't exactly dead - that was just an attention grabbing title. But Jenkins is becoming *legacy* - and in the world of sotware development that is like being on perpetual life support. Before I explain, let me tell you what I mean by *legacy*. For me, legacy means you are using a tool or processes because you feel like it is too difficult to switch to what you know is a better tool or process. Under that definition, deployments to VMs are legacy - being supplanted by containers; monolithic applications are legacy - being supplanted by microservices; and although perhaps not legacy, Docker is dying - being supplanted by containerd for managing container-lifecycle and Kubernetes for container orchestration. Those are just some examples, but you get the idea.

At its core, Jenkins is a monolithic Java based web application based on a project called Hudson which dates back to 2004. So what's wrong with that?

 - Jenkins is static and in the world of Cloud Native, static equals higher costs. Jenkins agents have slowly moved to be dynamic and ephemeral, but only very recently has Jenkins itself [moved towards dynamic and ephemeral](https://medium.com/@jdrawlings/serverless-jenkins-with-jenkins-x-9134cbfe6870).
 - The ability to configure Jenkins as-code is challenging to say the least - even with projects like the [Jenkins CasC Plugin](https://github.com/jenkinsci/configuration-as-code-plugin). This makes things like disaster recovery more challenging.
 - Jenkins is typically the SPOF for an organization's CD - if Jenkins goes down webhooks are missed, builds stop, deployments don't happen - CD stops.
 - The Jenkins UI leaves much to be desired - but this is less of an issue because in a native K8s CD world there shouldn't really be a UI for CD. But it is an issue if you are still manually creating Jenkins Freestyle jobs (please stop doing that).

But Jenkins won't disappear overnight or for a very long time. The reason why legacy Jenkins will be around for many more years is because of how slow many organizations are to move away from other legacy tools - for example taking 3 years or more for a large enterprise to migrate to a new version-control system and that version-control system is already considered to be legacy. There are large enterprise IT organizations that are still using Subversion or ClearCase or even [Concurrent Versions System](http://savannah.nongnu.org/news/?group=cvs). You will certainly need to keep using legacy Jenkins to support those legacy version-control systems and other legacy tools/processes -  because they aren't and won't be supported by native K8s CD. Legacy CD, Jenkins, is being supplanted by cloud native CD, with Jenkins X leading the way for truly native Kubernetes based CD - and that is a very good thing.

## Why is Native K8s CD important?

Before we get to the what, let's talk about why native K8s CD is important.

- **Standardization:** Kubernetes has become the defacto standard for container orchestration, the same thing is happening for Kubernetes CD Pipelines.
- **Lead, Not Follow:** Jenkins X is proving to be a leader in the K8s CD space by adopting and contributing to other key K8s CD related projects that it leverages to be the best comprehensive CD platform on K8s.
- **Performance:** Jenkins is great (and will still have a market for years to come), but it is based on architecture that is over decade old. Native K8s CD will be lighter, faster, stronger and Jenkins X will make it easy to use.

## What is Native K8s CD?

### So, what does it mean to be native K8s CD?
- First, the CD process itself must run on and be orchestrated by K8s. This by itself does not constitute native K8s CD - as it is [very easy to run legacy Jenkins on K8s](https://github.com/helm/charts/tree/master/stable/jenkins).
- Secondly, containers must be used to execute the steps of a CD pipline. Once again, this by itself does not constitute native K8s CD as legacy Jenkins has been using containers for CD pipeline steps since 2014 - as do many other CD tools like GitLab for example which is no more native K8s CD than legacy Jenkins.
- Finally, to be native K8s CD the tool must leverage native K8s resources or more specifically, must be built on top of [Custom Resource Definitions (CRDs)](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/). This allows you to use native Kubernetes tools like `kubectl` to interact with the tool. Legacy Jenkins does not fit this part of the native K8s CD definition. And actually, believe it or not, neither did Jenkins X - at least until recently.

### So why wasn't Jenkins X completely native K8s CD before now? 
Jenkins X runs on and is orchestrated by K8s. Jenkins X uses containers to execute the steps of CD pipelines. Jenkins X even leverages [its very own CRDs](https://github.com/jenkins-x/jx/blob/master/pkg/kube/crds.go) - but up until recently, it still relied on legacy Jenkins to actually execute the CD pipeline. And even with the very cool [Serverless Jenkins](https://jenkins-x.io/news/serverless-jenkins/) capabilities it still had legacy Jenkins as its engine to process pipelines.

### So what has changed recently to make Jenkins X truly native K8s CD? 
[Tekton](https://tekton.dev/). Tekton, along with Jenkins X, are two of the other initial projects that are part of the CDF. Tekton is an open source project that provides [100% K8s resources as a standard industry specification](https://github.com/tektoncd/pipeline/blob/master/docs/README.md) for end-to-end CD pipelines. And [Jenkins X has already integrated Tekton](https://jenkins-x.io/news/jenkins-x-next-gen-pipeline-engine/) in what will become the default way to process Jenkins X Next Gen pipelines - of course there will still be [a legacy Jenkins available for legacy pipelines in Jenkins X](https://github.com/jenkins-x-apps/jx-app-jenkins). But with Tekton, as the pipeline processing engine, Jenkins X is now truly native K8s CD.

### So why not just use Tekton? 
For today we will just leave it at a comparison of a [**Jenkins X Next Gen pipeline file**](#jenkins-next-gen-pipeline) versus the [**Tekton CRDs that Jenkins X generates**](#tekton-crds-generated-by-jenkins-x) for you. In a future post - and as the Tekton integration and Next Gen Pipeline becomes more stable, I will explore the Jenkins X Next Gen Pipelines and Tekton integration in more detail.

#### Jenkins Next Gen Pipeline 
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


