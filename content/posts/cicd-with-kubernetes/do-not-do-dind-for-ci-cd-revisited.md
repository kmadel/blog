---
title: Docker-in-Docker for CI - A Kubernetes Perspective
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","autoscaling","DIND","security"]
part: 4
date: 2018-07-18T23:09:15-04:00
draft: true
---
In 2015 Jérôme Petazzoni published an article entitled "Using Docker-in-Docker for your CI or testing environment? Think twice." The article basically describes how using Docker-in-Docker (or DIND) is not a good choice for a CI/CD workload for a number of different reasons - https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/. He promoted the concept of moutning the host the Docker socket as a volume in a Docker container to accomplish similar functionality but without the drawbacks of DIND - layer caching, privileged mode. In Part 4 of this CI/CD on Kubernetes series we will explore why it is a bad idea to use either of these two approaches. We will look at this from two different perspectives: security and infrastructure/performance. Finally, we will take a look at an alternative approach.

Securing Jenkins Agent with Kubernetes 

## What's Wrong with Docker-in-Docker (DIND)

Security - container must run with privileged flag enabled.
Performance - Layer caching is not shared across buids.

It used to be don’t do DIND for CI/CD - now it is don’t use the Docker socket for CI/CD on Kubernetes. Let’s just put security aside for a moment - it turns out that mounting the Docker socket is much worse in a Kubernetes based CI/CD environment than DIND. A Kubernetes cluster is made up of one or more worker nodes - and is some cases, the master node is also a worker node, but you shouldn’t do this. It is on these worker nodes where Kubernetes schedules and runs Pods - if you don’t know what a Pod is then read this and come back when you are done. When you mount the Docker socket to a Pod you are mounting the /var/run/docker.sock file into each and every container that makes up your Pod and when those containers run Docker commands against that socket they are acutally being executed by the Docker daemon running on the worker node where the Pod was scheduled. The Kubernetes scheduler has no clue that these other containers are running - they aren’t managed by Kubernetes, rather they are managed by the containers with access to the worker node Docker socket. This will result in all kinds of Kubernetes scheduling issues, especially on busy clusters.
Just move to Containerd??? can you mount the socket with containerd
What is the alternative? By using the Jenkins Kubernetes plugin for dynamic agents and the `container` Pipeline step you can run many of you CI/CD steps directly in containers build specifically and singularly for the purpose of executing that step or step(s), all managed within a Pod that Kubernetes manages.
Review Jenkins Kubernetes plugin vs other container based solutions for agents.
PodSecurityPolicy
Don’t allow mounting Docker socket
Don’t allow privileged containers


Other security vectors
Enforce use of specific registries - for example don’t allow DockerHub
IAM on worker node
