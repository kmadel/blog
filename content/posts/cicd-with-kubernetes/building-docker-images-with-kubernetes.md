---
title: Building Docker Images with Kubernetes
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","autoscaling","DIND","security"]
part: 4
date: 2019-01-05T23:09:15-04:00
draft: true
---
Way back in 2013, before Google had even introduced Kubernetes to the world, Docker was making containers popular. At the same time, continuous integration (CI) was becoming a common best practice for application development. The use of Docker containers with CI started to be adopted as a best practice to manage CI tools - compilers, testing tools, security scans, etc. But it was new and there weren't a lot of best practices to adopt - it was more of a 'go figure it out' process. Building  containers and pushing them to container registry was one very important aspect of CI with and for containers. But what was the best way to do it? 

Fast forward a couple of years to September of 2015. Jérôme Petazzoni published an article entitled ["Using Docker-in-Docker for your CI or testing environment? Think twice."](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) The article basically describes how using Docker-in-Docker (DIND) is not a good choice for a CI/CD workload for a number of different reasons. He promoted the concept of mounting the host Docker socket as a volume in a Docker container to accomplish similar functionality but without the drawbacks of DIND -  to include layer caching and requiring privileged mode. In Part 4 of this CI/CD on Kubernetes series we will explore why it is a bad idea to use either of these two approaches for building and pushing container images from Kubernetes. We will look at this from two different perspectives: security and infrastructure/performance. Finally, we will take a look at an alternative approach.

Securing Jenkins Agent with Kubernetes 

## What's Wrong with Docker-in-Docker (DIND)

Security - container must run with privileged flag enabled.
Performance - Layer caching is not shared across buids.

It used to be: Doing DIND for CI/CD? Think twice. But in a Kubernetes world it should be think twice if you are mounting the Docker socket for CI/CD. Let’s just put security aside for a moment - it turns out that mounting the Docker socket is worse in a Kubernetes based CI/CD environment than is DIND. A Kubernetes cluster is made up of one or more worker nodes - and in some cases, the master node is also a worker node (although you probably shouldn't use the master node as a worker node). It is on these worker nodes where Kubernetes schedules and runs Pods - if you don’t know what a Pod is then read [this](https://kubernetes.io/docs/concepts/workloads/pods/pod/) and come back when you are done. When you mount the Docker socket to a Pod you are mounting the `/var/run/docker.sock` file into each and every container that makes up your Pod and when those containers run Docker commands against that socket they are acutally being executed by the Docker daemon running on the worker node where the Pod was scheduled. The Kubernetes scheduler has no clue that these other containers are running - they aren’t managed by Kubernetes, rather they are managed by the Pod container(s) with access to the worker node Docker socket. This will result in Kubernetes scheduling issues, especially on busy clusters. And one of the main reasons to use Kubernetes in the first place is because of its robust orchestration and scheduling capabilities.

### Security
DIND has [always required that the privileged flag for the container be enabled](https://blog.docker.com/2013/09/docker-can-now-run-within-docker/). But mounting the Docker socket has never really been much more secure, and relied on the use of Docker daemon instances dedicated to CI/CD to be secure in the sense that you weren't to woried if a CI/CD job hosed the Docker daemon instance. 

On Kubernetes, DIND still requires the privileged flag to be enabled. But again, mounting the Docker socket is even worse as it requires the Pod privileged flag to be enabled ****need to check this****.

"While this is a 9.8/10 on the "bad" scale for a single-purpose cluster just doing Jenkins things, it's an 11/10 on the "bad" scale for a cluster with multiple workloads that need isolation. e.g. Production is just a namespace away from the namespace where Jenkins is running jobs.
The most practical exploitation vector is a curious/malicious developer. If they can modify a Jenkinsfile to insert direct calls to run docker commands as one of the build steps, then they can become root on the node where that job lands. If they gain access to the underlying host as root, there are reasonably straightforward methods to escalate to gain access to the entire cluster.

Many cluster operators don't have the visibility in place to properly detect this kind of activity and separate it from legitimate Jenkins/Docker job runs, so the escape is likely to go unnoticed for a while.

If you go down this path, the net result is "anyone who can modify a Jenkinsfile has a way to become root in my entire cluster". If you're cool with that implication, proceed. If not, don't."


Just move to Containerd??? can you mount the socket with containerd
What is the alternative? By using the Jenkins Kubernetes plugin for dynamic agents and the `container` Pipeline step you can run many of you CI/CD steps directly in containers build specifically and singularly for the purpose of executing that step or step(s), all managed within a Pod that Kubernetes manages.
Review Jenkins Kubernetes plugin vs other container based solutions for agents.
PodSecurityPolicy
Don’t allow mounting Docker socket
Don’t allow privileged containers


Other security vectors
Enforce use of specific registries - for example don’t allow DockerHub
IAM on worker node

## Building Container Images without Docker
Or more specifically, building container images without the Docker daemon. 

### Kaniko


### Other Solutions