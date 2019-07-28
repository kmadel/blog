---
title: "Prow: Keeping Kubernetes CI/CD Above Water"
series: ["Native Kubernetes Continuous Delivery"]
tags: ["CDF","CI","CD","Tekton","Kubernetes", "Prow","Jenkins X", "Native K8s CD"]
date: 2019-04-15T06:00:15-04:00
photo: "/photos/native-kubernetes-continuous-delivery/genova-prow.jpg"
photoCaption: "Small Fishing Boat, Genova, Italy<br>Photograph by Kurt Madel ©2019"
exif: "SONY RX-100 ISO 125 15.45mm ƒ/5.6 1/640"
part: 3
draft: false
---
{{< load-photoswipe >}}
If you are doing CI and/or CD at scale and you aren't leveraging Native Kubernetes Continuous Delivery (Native K8s CD) then you are just ~~doing it wrong~~ [missing out on a better way](../native-k8s-cd) - plain and simple. And if there is one Kubernetes project that has been at the forefront of Native K8s CD and best exemplifies the why and the how of what makes Kubernetes such an excellent platform for executing CI/CD at scale - it is Prow. [Prow](https://github.com/kubernetes/test-infra/tree/master/prow) is a Native Kubernetes CI/CD system and as far as its name goes, Prow didn't go the Greek route for its K8s-themed name, rather it went the nautical route with *prow* meaning ***the portion of a ship's bow (the front end of a ship) above water*** - and Prow has been keeping K8s CI/CD above water for several years now. And others have taken notice, resulting in Prow becoming an important tool for a number of high-profile Kubernetes-centric projects. [Prow](https://github.com/tektoncd/pipeline/blob/master/infra/prow/config.yaml) is an important tool in the toolbox of another Native K8s CD project called [Tekton](https://tekton.dev/), going the Greek route for its name which means carpenter, that I wrote about in a [previous post of this Native K8s CD series](../tekton-standardizing-native-kubernetes-cd/). Tekton, along with a number of other important K8s projects, use Prow for their own CI/CD automation even though there are a number of other options. So what exactly is Prow and why should you care? Before we dive into that, I believe it is important to understand the origin and evolution of Prow to provide additional insight into why K8s is such an excellent platform for CI/CD. 

## Where did Prow come from?
When it came to CI/CD and the creation of Prow, Kubernetes was scratching its own itch with a need to execute over 10,000 CI/CD jobs per day across over 100 repositories of code in several GitHub Organizations - and other tools just weren't getting the job done. So, the [Kubernetes Testing Special Interest Group (sig-testing)](https://github.com/kubernetes/community/blob/master/sig-testing/README.md) created their own tools and services, to include Prow. Prow is currently part of the [Kubernetes test-infra project](https://github.com/kubernetes/test-infra) and is one of several [Kubernetes Testing Special Interest Group (sig-testing)](https://github.com/kubernetes/community/tree/master/sig-testing) top-level projects. At the time of this post, Prow was not even in its own GitHub repository, rather it was [just a sub-directory](https://github.com/kubernetes/test-infra/tree/master/prow) (and previously named [ciongke](https://github.com/kubernetes/test-infra/pull/835))in the *kubernetes/test-infra* repository. And partly because of the importance of Prow to the K8s project as a whole and also because there is so much interest from other K8s related projects, Prow continues to add features and flourish. Another testament to Prows continued growth is how top-level [**test-infra**](https://github.com/kubernetes/test-infra) components continue to be replaced with Prow components - with one of the latest examples of this being [the replacement of Gubernator with Spyglass](https://github.com/kubernetes/test-infra/pull/11302).

## What is Prow?
Prow is a lot - certainly more than I can cover in one post - but I will do my best. Prow's [GitHub README](https://github.com/kubernetes/test-infra/blob/master/prow/README.md) describes it as a *system*, and a very complex multi-faceted system it is at that. Prow is built on a container based microservice architecture and consists of a number of single purpose components - many integrating with one another while others are more or less standalone. The Kubernetes Testing SIG describes Prow as *"a CI/CD system built on Kubernetes for Kubernetes that executes jobs for building, testing, publishing and deploying."* However, that description does not highlight perhaps the most important inferred capability of Prow - a capability that is at the heart of *best-of-breed* CI/CD automation tools - that capability is automation that starts with code commits - and in the case of Prow it starts with a scalable stateless microservice called [`hook`](https://github.com/kubernetes/test-infra/blob/master/prow/cmd/README.md#core-components)that triggers native K8s CI/CD jobs (among a number of things that `hook` does via [plugins](https://github.com/kubernetes/test-infra/tree/master/prow/plugins)). It is this GitHub automation capability that has been one of the key reasons why other K8s projects have adopted Prow for their own CI/CD. But Prow is more than just GitHub webhook automation and CI/CD job execution. Prow is also:

* Comprehensive GitHub Automation
  * ChatOps via [simple `/foo` commands](https://prow.k8s.io/command-help)
  * Fine-grained GitHub policy and permission management via [`OWNERS` files](https://github.com/kubernetes/community/blob/master/contributors/guide/owners.md)
  * GitHub PR/merge automation - [`tide`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/tide)
  * GitHub API request cache to minimize impact of GitHub API limits - [`ghProxy`](https://github.com/kubernetes/test-infra/tree/master/ghproxy)
  * GitHub Organization and repository membership and permissions management - [`peribolos`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/peribolos)
  * GitHub labels management - [`label` plugin](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/label)
  * GitHub branch protection configuration - [`branchprotector`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/branchprotector)
  * GitHub release notes management - [`releasenote`](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/releasenote)
  * [Scalable, cacheable](https://github.com/kubernetes/test-infra/blob/master/prow/scaling.md#github-api-cache) 
  * GitHub [bot](https://github.com/kubernetes/test-infra/blob/master/prow/getting_started_deploy.md#bot-account) with [Prow's bot being an active GitHub user since 2016](https://github.com/k8s-ci-robot)
* Multi-engine CI/CD Job Execution - [`plank`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/plank)
* CI/CD Reporting - [`crier`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/crier)
* CI/CD Dashboards for viewing job history, merge status and more - [`deck`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/deck)
* Pluggable Artifact Viewer - [`Spyglass`](https://github.com/kubernetes/test-infra/tree/master/prow/spyglass)
* Promethus Metrics for monitoring and alerting - [`metrics`](https://github.com/kubernetes/test-infra/tree/master/prow/metrics)
* Config-as-Code for Its Own Configuration - [`updateconfig`](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/updateconfig)
* And I am sure I am still missing a bunch of things - like [`cats`](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/cat)
 and [`dogs`](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/dog) 

### Comprehensive GitHub Automation
*Prow is like steroids for GitHub automation.*

#### Prow is a GitHub Webhook Listener
This may be one of the most important capabilities of Prow and most of Prow's other components depend on it in one way or another. Prow listens for GitHub webhooks via the `hook` component that in turn triggers one or more configured [plugins](https://prow.k8s.io/plugins) to include the [`trigger` plugin](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/trigger) that triggers Native K8s CI/CD jobs for GitHub PR events, and the `/test` and `/retest` ChatOps commands.

Many non-Native K8s CI/CD platforms support GitHub webhooks but in many cases they are not highly fault-tolerant, not horizontally scalable and not stateless - so they become a single-point-of-failure (SPoF) for GitHub webhooks. If a GitHub webhook is sent while such a platform is down it will be completely missed - so you might miss a merge to your deployment branch and miss a mission critical change that needs to be deployed to production.

#### Prow ChatOps
ChatOps are an important component of Prow's GitHub webhook support - Prow doesn't just listen for pushes to branches, but listens for a number of more sophisticated GitHub events, like comments on issues and PRs using ChatOps commands. Prow ChatOps allows you to do everything from approving a PR with the [`/approve` command](https://prow.k8s.io/command-help#approve) to telling a [`/joke`](https://prow.k8s.io/command-help#joke). Many of the [Prow `hook` plugins](https://prow.k8s.io/plugins) provide ChatOps commands that streamline and automate repository contributors' interactions with GitHub PRs and issues - so developers can focus more on the code and not GitHub process and UI. The [complete list of supported ChatOps commands](https://prow.k8s.io/command-help) is available as [one of the views of the `deck` component](https://prow.k8s.io/command-help). 

#### Prow Provides More Granular Directory Level Permissions in GitHub
Prow allows you to manage ownership/permissions at a more granular level than GitHub does. Prow allows you to create an [`OWNERS` file](https://github.com/kubernetes/community/blob/master/contributors/guide/owners.md) at the repository level, but also within any repository sub-directory. This allows you to manage who can do what at the directory level instead of just the top-level repository level. The directory specific `OWNERS` files are utilized by a number of `hook` plugins and ChatOps commands - for example the [`blunderbluss` plugin automatically assigns PR reviewers](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/approve/approvers#blunderbuss-and-reviewers) based on `OWNERS` files most relevant to the file(s) being committed - but the automated reviewer selection is more sophisticated than just using the most relevant/closest `OWNERS` file - it also [weights reviewers based on the number of lines of code that changed and in what files](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/approve/approvers#blunderbuss-selection-mechanism) - once again highlighting the incredible breadth and depth of Prow's GitHub automation.

#### Prow is Automated GitHub Merges
Prow's [`tide`](https://github.com/kubernetes/test-infra/tree/master/prow/tide) component automates the bulk testing of PRs and automatically merges multiple PRs. It [replaced another `test-infra` top-level project called `mungegithub`](https://github.com/kubernetes/test-infra/commit/9d38f05fab58d5bf49921959dd36004db7cde6ac). In addition to automated testing and merging of PRs - at scale, tide also provides Promethus metrics and serves live data to Deck - the Prow reporting dashboard.

#### Prow Provides GitHub Organization Settings and Team Membership Management
Prow allows you to manage GitHub Organization settings and team membership with Config-as-Code (CasC) so the source of truth for your Organization configuration is in one easily auditable file, rather than relying on one or more people making manual changes via the GitHub UI or manual API calls. The [`peribolos` component](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/peribolos) provides automated auditable support for managing these GitHub settings with a CasC [`org.yaml` file](https://github.com/kubernetes/org/blob/master/config/kubernetes/org.yaml).

#### Prow Provides GitHub Project Management
Prow allows you to manage GitHub projects by assigning and/or moving GitHub issues/PRs to project columns. The [`project` plugin](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/project) provides the capability for you to add an issue or PR to a GitHub project board and column.

### Prow Provides Config-as-Code for Its Own Configuration
Prow is stateless - storing its configuration in a K8s `ConfigMap` - but you don't have to manually create or update the Prow `ConfigMap`. Prow, of course, provides a way to automate its own configuration by creating the different configuration files in GitHub.
Configuring Prow for your own GitHub Organization/Repository is based on a [`prow/config.yaml`](https://github.com/kubernetes/test-infra/blob/master/prow/config.yaml) file and you can use Prow itself to update it on PR merges.

Prow provides components to manage its own configuration with GitHub - to include the ability to test configuration changes before you apply them. Create a new PR with configuration changes and it will trigger an automated configuration test. Merge the PR to the `master` branch and Prow will update its K8s `ConfigMap` to match the configuration in your GitHub repository.

- [`updateconfig` plugin](https://github.com/kubernetes/test-infra/tree/master/prow/plugins/updateconfig ) allows Prow to update K8s `ConfigMaps` when Prow configuration files in a repository change.
- [`checkconfig` component](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/checkconfig) loads the Prow configuration given with `--config-path`, `--job-config-path` and `--plugin-config` in order to validate it. Use `checkconfig` as a pre-submit for any repository holding Prow configuration to ensure that check-ins do not break anything.
- [`config-bootstrapper`](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/config-bootstrapper) is a component used to bootstrap a configuration that would be incrementally updated by the `updateconfig` plugin.

###  Prow is a CI/CD Job Executor
Prow is very modular and triggers jobs of different types, triggered in different ways for different execution engines and provides dashboards to track job status, job artifacts and merge status among other things.

<p>

* Job Types - specifies how the job is triggered.
  * presubmit - runs on unmerged PRs.
  * postsubmit - runs on each new commit.
  * periodic - runs on a time-basis, unrelated to git changes.
  * batch - tests multiple unmerged PRs at the same time.

<p>

* Job States - specifies whether the job is running or not, the status of the job.
  * triggered - means the job has been created but is not yet scheduled.
  * pending - means the job is scheduled but not yet running.
  * success - means the job completed without error (exit 0).
  * failure - means the job completed with errors (exit non-zero).
  * aborted - means that prow killed the job early (new commit pushed, perhaps).
  * error - means the job could not schedule (bad config, perhaps).

<p>

* Execution Engines/Job Agents - specifies the controller (such as plank or jenkins-agent) that runs the job.
  * kubernetes - Prow will create a pod to run this job (default).
  * jenkins - Prow will schedule the job on an external static Jenkins instance.
  * knative-build - Prow will schedule the job via a [Knative build-crd resource](https://github.com/kubernetes/test-infra/blob/master/prow/cmd/build/main.go).
  * tekton ([coming soon](https://github.com/kubernetes/test-infra/pull/11888)) - Prow will schedule the job via a Tekton CRD based [PipelineRun](https://github.com/tektoncd/pipeline/blob/master/docs/pipelineruns.md).

<p>

* Reporting and Dashboards
  * [Deck](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/deck) provides:
      * [A job status dashboard](https://prow.k8s.io/)
      * [PR status](https://prow.k8s.io/pr)
      * [PR merge status via Tide](https://prow.k8s.io/tide)
      * [PR merge history via Tide](https://prow.k8s.io/tide-history)
      * [Prow ChatOps command reference](https://prow.k8s.io/command-help)
      * [Prow Plugin documentation](https://prow.k8s.io/plugins)
  * [Crier](https://github.com/kubernetes/test-infra/tree/master/prow/crier) detects changes to ProwJob CRDs and notifies configured clients to include: 
      * GitHub status context
      * Gerrit comment
      * Pubsub message
  * Artifact viewer with Spyglass

### Who is using Prow?
Prow provides a [list of other K8s organizations/projects using Prow](https://github.com/kubernetes/test-infra/tree/master/prow#prow-in-the-wild). But what is very interesting is the other Native K8s CD projects that leverage Prow for their own CI/CD:

- Knative - not only [provides one of the CI/CD job execution engines for Prow](https://github.com/kubernetes/test-infra/tree/master/prow/cmd/build), it also [uses Prow for its own CI](https://github.com/knative/test-infra/tree/master/ci/prow).
- Tekton - currently [using Prow to build and test Tekton Pipeline](https://github.com/tektoncd/pipeline/tree/master/infra/prow).
- Jenkins X - [uses Prow for its own CI/CD](https://github.com/jenkins-x/prow-config). But it also takes Prow usage to a whole other level by [seamlessly integrating Prow with the original Serverless Jenkins X and now for Jenkins X with Tekton](https://github.com/jenkins-x/jx/tree/master/pkg/prow) - [more about this in the next post](../jenkins-x-goes-native/).
- Istio - the service mesh used by Knative to support continuous deployment (among others of course), [integrated Prow early on its existence](https://github.com/istio/test-infra/pull/253).

## What's next?
So as you can see, and as I already mentioned above, Prow is a lot! And I definitely didn't discuss all of Prow's features in this post. But I would also like to share some things I would like to see next for Prow:

- I would like to see Prow become a top-level project/repository under kubernetes-sig GitHub Organization. See https://github.com/kubernetes/test-infra/issues/11782 for some details. 
- I would like to see support for other Git providers - BitBucket, GitHub Enterprise and even generic git support. See:
  - https://github.com/kubernetes/test-infra/issues/10146
- I would like to see Prow support Tekton as a Prow agent type/execution engine. See:
  - https://github.com/tektoncd/pipeline/issues/537 
  - https://github.com/jenkins-x/test-infra/blob/pipeline5/prow/apis/prowjobs/v1/types.go#L76
- Prow originated as a Google engineering project as displayed by Prow's GCP slant. I would like to see better support for AWS and Azure.
- I would like to see Prow become a [Continuous Delivery Foundation](https://cd.foundation/) project.

Most of all, I want to see you keep your own CD above water [by using Prow for your own GitHub automation]((https://github.com/kubernetes/test-infra/blob/master/prow/getting_started_deploy.md)). But before you do that, you may want to wait for my [next post in this series](../jenkins-x-goes-native/) where I will explain how effortless it can be for you to realize the advantages of Prow for your own application delivery by using Jenkins X for your CD.





