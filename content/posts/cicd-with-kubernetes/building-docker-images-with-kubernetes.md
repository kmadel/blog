---
title: Mounting the Docker Socket for your CI or testing environment? Think twice.
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","autoscaling","DIND","security"]
part: 4
date: 2019-01-05T23:09:15-04:00
draft: true
---
Back in 2013, before Kubernetes was a thing, Docker was making Linux containers popular. At the same time, continuous integration (CI) was becoming a common best practice for application development. The use of Docker containers with CI began to be adopted as an easy and efficient way to manage CI tools - compilers, testing tools, security scans, etc. But it was new and there weren't a lot of best practices to adopt - it was more like 'go figure it out'. And early on, one very important aspect of using containers for CI/CD was using containers to build container images and pushing those images to a Docker registry - but again, this was all very new, and a lot of people didn't really know what they were doing and there wasn't a manual.

Fast forward a couple of years to September of 2015. Jérôme Petazzoni published an article entitled ["Using Docker-in-Docker for your CI or testing environment? Think twice."](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) The article basically describes how using Docker-in-Docker (DIND) is not a good choice for a CI/CD workload for a number of different reasons - it is definitely an article still worth a read. He promoted the concept of mounting the host Docker socket as a volume in a Docker container to accomplish similar functionality but without the drawbacks of DIND -  to include layer caching and requiring [privileged mode](https://blog.docker.com/2013/09/docker-can-now-run-within-docker/). In Part 4 of this CI/CD on Kubernetes series we will explore why it is a bad idea to use either of these two approaches for building and pushing container images from Kubernetes. We will look at this from two different perspectives: security and infrastructure/performance. Finally, we will take a look at an alternative approach.

## What's Wrong with Docker-in-Docker (DIND)

Security - container must run with privileged flag enabled.
Performance - Layer caching is not shared across buids.

So it used to be that doing DIND for CI/CD was a bad idea. But in a Kubernetes world you should think twice if you are mounting the Docker socket for CI/CD. Let’s just put security aside for a moment - it turns out that mounting the Docker socket has its own set of detrimental issues in a Kubernetes based CI/CD environment. A Kubernetes cluster is made up of one or more worker nodes and it is on these worker nodes where Kubernetes schedules and runs [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/). When you mount the Docker socket to a Pod you are mounting the `/var/run/docker.sock` file into each and every container that makes up your Pod and when those containers run Docker commands against that socket they are acutally being executed by the Docker daemon running on the worker node where the Pod was scheduled. The Kubernetes scheduler has no way to track that these other containers are running - they aren’t managed by Kubernetes, rather they are managed by the Docker daemon running on the node wher the Pod gets scheduled. This will result in Kubernetes scheduling issues, especially on busy clusters. And one of the main reasons to use Kubernetes in the first place is because of its robust orchestration and scheduling capabilities, so why would you want to circumvent that for CI/CD? 

### Security
**Disclaimer:** I am not a security expert. But in simplistic terms, increasing security goes hand in hand with reducing the attack surface or attack vectors. It is no different with running Jenkins on Kubernetes. If the Docker socket is exposed to Jenkins jobs then a curious/malicious developer can modify a Jenkinsfile to run docker commands as one of the build steps, then they can become root on the node where that job lands. If they gain access to the underlying host as root, there are reasonably straightforward methods to escalate privileges and gain access to the entire cluster.

DIND has always [required that the privileged flag for the container be enabled](https://blog.docker.com/2013/09/docker-can-now-run-within-docker/). But mounting the Docker socket has never really been much more secure, and relied on the use of dedicated Docker daemon instances to isolate CI/CD workloads from other container workloads - like production applications. 

On Kubernetes, DIND still requires the privileged flag to be enabled. But again, mounting the Docker socket is even worse as in many cases it also requires the Pod privileged flag to be enabled. And since the Docker daemon has to run as root, giving a container direct access to the node's Docker daemon via

"While this is a 9.8/10 on the "bad" scale for a single-purpose cluster just doing Jenkins things, it's an 11/10 on the "bad" scale for a cluster with multiple workloads that need isolation. e.g. Production is just a namespace away from the namespace where Jenkins is running jobs.
The most practical exploitation vector is a curious/malicious developer. If they can modify a Jenkinsfile to insert direct calls to run docker commands as one of the build steps, then they can become root on the node where that job lands. If they gain access to the underlying host as root, there are reasonably straightforward methods to escalate to gain access to the entire cluster.

Many cluster operators don't have the visibility in place to properly detect this kind of activity and separate it from legitimate Jenkins/Docker job runs, so the escape is likely to go unnoticed for a while.

If you go down this path, the net result is "anyone who can modify a Jenkinsfile has a way to become root in my entire cluster". If you're cool with that implication, proceed. If not, don't."

### Performance
Besides security implications, performance was another big reason that mounting the Docker socket became the preffered solution for building and pusing Docker images. DIND does not cache image layers.

### Block the Use of DIND and the Docker Socket
So, for some, the drawbacks of mounting the Docker socket for CI/CD on Kubernetes are significant enough that they [recommended going back to DIND](https://applatix.com/case-docker-docker-kubernetes-part-2/). But we have already dismissed both DIND and mounting the Docker socket as approaches for CI/CD on Kubernetes.

Just move to Containerd??? can you mount the socket with containerd

GKE has beta support for containerd nodes.
What is the alternative? By using the Jenkins Kubernetes plugin for dynamic agents and the `container` Pipeline step you can run many of you CI/CD steps directly in containers build specifically and singularly for the purpose of executing that step or step(s), all managed within a Pod that Kubernetes manages.
Review Jenkins Kubernetes plugin vs other container based solutions for agents.
PodSecurityPolicy
Don’t allow mounting Docker socket
Don’t allow privileged containers


Other security vectors
Enforce use of specific registries - for example don’t allow DockerHub
IAM on worker node

## Building Container Images without Docker
Or more specifically, building container images without the Docker daemon. So

### Kaniko

#### Kaniko with AWS and the ECR
The Kaniko instructions tell you to create a Kubernetes secret for your `~/.aws/credentials` but many, if not most, organizations don't allow you to use aws credentials this way. 

Use kube2iam 
With kops you can add additional IAM policies to be applied to all Kubernetes nodes.

```
apiVersion: kops/v1alpha2
kind: Cluster
metadata:
  creationTimestamp: 2018-04-29T01:06:51Z
  name: workshop.beedemo.net
spec:
  additionalPolicies:
    node: |
      [
        {
        "Effect": "Allow",
        "Action": ["ecr:*"],
        "Resource": ["*"]
        }
      ]
```

### Other Solutions