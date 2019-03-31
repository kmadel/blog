---
title: "Prow: Keeping Kubernetes CI/CD Above Water"
series: ["Native Kubernetes Continuous Delivery"]
tags: ["CDF","CI","CD","Tekton","Kubernetes", "Prow","Jenkins X", "Native K8s CD"]
date: 2019-03-05T21:00:15-04:00
photo: "/photos/k8s-cd/prow-uss-wisconsin.jpg"
photoCaption: "USS Wisconsin, Norfolk, VA<br>Photograph by Kurt Madel ©2016"
exif: "Apple iPhone 6 Plus ISO 32 ƒ/2.2 1/842"
part: 3
draft: true
---
{{< load-photoswipe >}}
Native Kubernetes CD is becoming important. [Prow](https://github.com/kubernetes/test-infra/tree/master/prow) is a Kubernetes based CI/CD system and is becoming an important platform for a number of high-profile projects in the Kubernetes ecosystem. Rather than go the Greek route for a Kubernetes-centric name, Prow went the nautical route for a name with prow meaning the portion of a ship's bow (the front end of a ship) above water. I recently wrote about [Tekton](https://kurtmadel.com/posts/cicd-with-kubernetes/tekton-standardizing-native-kubernetes-cd/) and Tekton itself [uses Prow](https://github.com/tektoncd/pipeline/blob/master/infra/prow/config.yaml). Before we take a look at what exactly Prow is let's look at where Prow came from. 

## Where did Prow come from?
Kuberenets was scratching their own itch in regards to testing several hundred repositories of code across several GitHub Organizations and other tools weren't getting the job done. So, the [Kubernetes Testing Special Interest Group (sig-testing)](https://github.com/kubernetes/community/blob/master/sig-testing/README.md) created their own tools and services, to include Prow. Prow is currently part of the [Kubernetes test-infra project](https://github.com/kubernetes/test-infra) and is part of the [Kubernetes Testing Special Interest Group (sig-testing)](https://github.com/kubernetes/community/tree/master/sig-testing) with its genesis dating back to early 2016. At the time of this post, Prow was not even in its own GitHub repository, rather it was [just a sub-folder](https://github.com/kubernetes/test-infra/tree/master/prow) (and previously named [ciongke](https://github.com/kubernetes/test-infra/pull/835))in the *kubernetes/test-infra* repository that is one of the primary sig-testing projects. 

Kubernetes 5 GitHub Orgs and over 125 repositories. 10,000 CI jobs per day sometimes - probably more now as that was 8 months ago. https://kubernetes.io/blog/2018/08/29/the-machines-can-do-the-work-a-story-of-kubernetes-testing-ci-and-automating-the-contributor-experience/

## What is Prow?
Prow is a lot. Prow's [GitHub README](https://github.com/kubernetes/test-infra/blob/master/prow/README.md) describes it as a system, but Prow is more of a platform than it is any specific tool with specific functionality. The Kubernetes Testing SIG describes it as "a CI/CD system built on Kubernetes for Kubernetes that executes jobs for building, testing, publishing and deploying." But another very important aspect of Prow is the GitHub automation it provides - and this is the capability that has been a big driver of other tools reason for leveraging Prow. And then there are a whole number of other features and components on top of that.

### Prow is a GitHub Webhook Processor
This may be one of the most recognized features of Prow. Prow listens for for GitHub webhooks via the `hook` component that in turn triggers one or more configured [plugins](https://prow.k8s.io/plugins) with the `trigger` plugin being one of the most important Prow plugins. 

May not seem much better than Jenkins - and afterall, Jenkins has support for webhooks for a number for SCM tools - not just GitHub.

### Prow Triggers *Jobs* 
Prow is very modular - trigger jobs of different types, trigger in different ways and different types of jobs. 

Not sure if they are adding any more execution platforms but adding some soon, but very modular.

#### ProwJob
**Job Types** - specifies how the job is triggered.
* presubmit - runs on unmerged PRs.
* postsubmit - runs on each new commit.
* periodic - runs on a time-basis, urelated to git changes.
* batch - tests multiple unmerged PRs at the same time.

**Job States** - specifies whether the job is running or not.
* tirggered - means the job has been created but is not yet scheduled.
* pending - means the job is scheduled but not yet running.
* success - means the job completed without error (exit 0).
* failure - means the job completed with errors (exit non-zero).
* aborted - means that prow killed the job early (new commit pushed, perhaps).
* error - means the job could not schedule (bad config, perhaps).

**Job Agents** - specifies the controller (such as plank or jenkins-agent) that runs the job.
* kubernetes - means prow will create a pod to run this job (default).
* jenkins - means prow will schedule the job on Jenkins.
* knative-build - means prow will schedule the job via a build-crd resource.

**Utility Images** - pull specs for the utility images to be used for a job.
* clonerefs
* initupload
* entrypoint
* sidecar



ProwJobs support many different:
* Job types
* Triggering mechanisms
* Execution platforms
* Reporting sinks

Most common ProwJob:
* Presubmit - GitHub PR
  * GitHub webhook payload piped to /hook endpoint via ingress
  * hook is comprised of a number of plugins
    * trigger plugin reacts to a GitHub event or ChatOps like `/test-all`
    * hook plugins can interact with GitHub, K8s, Slack
    * trigger plugin determines which jobs to run based on the config.
    * Creates a new ProwJob CRD for each job
    * CRDs allow storing state ini the K8s API server
    * enter Plank - syncs ProwJob CRDs with Pods to execute the jobs, syncing from API server and synchronizing across build cluster(s)
      * Typical lifecycle
        * New PJ without a pod -> create pod
        * Running PS wit completed pod -> complete the PJ (pass/fail)
        * Complete PJ -> ignore
    * Sinker - cleans up completed ProwJobs 
      * historic resutls are uploaded to GCS
    * Crier - detects changes to ProwJobs CRDs and notifies reporting clients
      * listens for ProwJob changes
      * GitHub Reporter - GitHub status context
      * Gerrit Reporter
      * PubSub Reporter
      * Slack Reporter?
* Triggered via Prow ChatOps `/test all`
* Executed by Kubernetes Pod
* Reported as GitHub status context on PR

"If you have some terrible reason to you could run your job on Jenkins using Prow." - Cole Wagner, Testing SIG

Webhook

[Prow Plugin Catalog](https://prow.k8s.io/plugins)
[Prow Command Help](https://prow.k8s.io/command-help)

ChatOps enabled GitOps. ChatOps via `/foo` style commands.

Spyglass taking place of gubernator for Prow artifact browsing.

Jenkins X

vs Knative eventing
vs Tekton triggers

trigger plugin determines what Prow jobs to run based on configuration.

CRDs provide the ability to store state in the Kubernetes API server.

hook -> trigger -> plank -> sinker -> crier

Sinker is GC for ProwJobs - completed ProwJob CRDs are removed after 2 days and completed Pods are removed after 30 minutes.

Crier detects changes to ProwJob CRDs and notifies configured clients to possibly include:
* GitHub status context
* Gerrit comment
* Pubsub message

### How do you configure Prow?
Configuring Prow for your own GitHub Organization/Repository is based on the prow/config.yaml file based configuration.

"`updateconfig` allows prow to update configmaps when files in a repo change." from https://github.com/kubernetes/test-infra/tree/master/prow/plugins/updateconfig 

"`checkconfig` loads the Prow configuration given with --config-path, --job-config-path and --plugin-config in order to validate it. Use checkconfig as a pre-submit for any repository holding Prow configuration to ensure that check-ins do not break anything." from https://github.com/kubernetes/test-infra/tree/master/prow/cmd/checkconfig 

## Why is Prow important?
GitHub webhooks. Did you read the above section on ***What is Prow*** - that is just the tip of the iceberg. Prow is a lot more than can fit in one post such as this. Commit based CI and CD.

### Some Projects that Use Prow
OpenShift
Prometheus
Knative
Kubeflow
Istio, the service mesh used by Knative (among others of course), [integrated Prow early on its existence](https://github.com/istio/test-infra/pull/253).
Tekton - currently using Prow to trigger Tekton jobs.
Jenkins X - uses Prow for the original Serverless Jenkins X and now for the Jenkinsless Jenkins X with Tekton.

Others?

## What's next?
Top-level project/repository under kubernetes-sig GitHub Organziation. See https://github.com/kubernetes/test-infra/issues/11782 for some details. 
Support for other Git providers - BitBucket, GitHub Enterprise, generic git support?

Support for Tekton as an Prow agent type - see 
- https://github.com/tektoncd/pipeline/issues/537 
- https://github.com/jenkins-x/test-infra/blob/pipeline5/prow/apis/prowjobs/v1/types.go#L76
- 

Currently only pushes logs to GCS - looking to support AWS and Azure.

Like Tekton, Prow's genesis was from Google engineers and it shows.

CD - that is the delivery of software - is becoming as important as the software itself.

Prow will become a top-level Kubernetes project and in my opinion should become a CDF project.





