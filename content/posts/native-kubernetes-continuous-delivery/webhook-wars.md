---
title: The Webhook Wars
series: ["Native Kubernetes Continuous Delivery"]
tags: ["CI","CD","Tekton","Prow","Jenkins X","Kubernetes","Native K8s CD","git","webhooks"]
part: 6
date: 2019-09-06T09:09:15-04:00
photo: "/photos/native-kubernetes-continuous-delivery/secure-containers.jpg"
photoCaption: "Square Tower House, Ancient Pueblo Dwelling, Mesa Verde National Park, CO<br>Photograph by Kurt Madel ©2019"
exif: "SONY RX-100 ISO 125 37.1mm ƒ/5.6 1/160"
draft: true
---
I wrote about Prow in a previous post in this series and had high hopes that Prow would evolve into an important top-level project for the native Kubernetes continuous delivery ecosystem. Alas, that does not appear it will happen. Prow will remain an important component of the Kubernetes test-infra project but will limit its awesome GitOps with ChatOps support to GitHub. But native Kubernetes CD platforms need to support multiple Git hosting services - GitHub, Bitbucket, GitLab, Gitea, etc - if they are going to be more inclusive of a wider user base. But, supporting other Git hosting services besides GitHub will have to fall to projects outside of Prow and its parent Kubernetes test-infra project. And multiple projects have already started developing their own Prow replacements. Is this a good thing? Are these replacements better than Prow? Is there unnecessary duplicative work? Or have is this the beginning of **The Webhook Wars**.

## What's so important about webhooks?
Webhooks provide an important trigger for automated CI/CD and are absolutely necessary for a GitOps approach to CI/CD. When you create a pull request (merge request, etc) CD for your source repository should automatically be triggered. If your CD jobs aren't automatically triggered upon commits to Git then you are wasting precious time and resources. Prow has been a critical component in the evolution of native Kubernetes CD as reflected by the myriad Kubernetes related projects that use Prow - to include two very high profile native Kubernetes CD projects that will be the primary subjects of this post: Tekton and Jenkins X. Without Prow, serverless Jenkins would not exist. Without Prow you would have to manually run your Tekton Pipelineruns. Without Prow automated CD on Kubernetes would have stopped before it begun.

## What happened to Prow?
The test-infra project was not going to embrace the changes necessary to support multiple Git providers.

Only supports GitHub and has no interest in supporting other Git providers - GitHub Enterprise, BitBucket, Gitea, plain vanilla git and Gitlab - among others.

## So what's next?
Tekton vs Jenkins X vs ??

Lighthouse vs Foghorn vs Tekton triggers

## So who is going to win?
Duplicative effort.
Jenkins X typically embraces other tools and components, but sometimes moves too fast.