---
title: Is Your Microservice Turning Brown?
series: ["Cloud Native Architecture"]
tags: ["CI","CD","Hybrid-Cloud","Multi-Cloud","Anthos","Jenkins X","Kubernetes","Native K8s CD"]
part: 1
date: 2019-09-03T09:09:15-04:00
photo: "/photos/native-kubernetes-continuous-delivery/secure-containers.jpg"
photoCaption: "Square Tower House, Ancient Pueblo Dwelling, Mesa Verde National Park, CO<br>Photograph by Kurt Madel ©2019"
exif: "SONY RX-100 ISO 125 37.1mm ƒ/5.6 1/160"
draft: true
---
## The Rise of Microservices
Microservice architecture has been one of the main areas of focus for [Greenfield](https://en.wikipedia.org/wiki/Greenfield_project) application delivery in the last half-decade plus. Developing microservice based APIs has coincided with the rise of containers and ephemeral cloud infrastructure, and has required a new way to think about deployment and how applications discover each other, among a number of other things.

But microservices don't take full advantage of containerization and ephemeral cloud infrastructure. Although loosely coupled, a *typical* microservice is always running - meaning they are always incurring infrastructure expenses, even when they are not being utilized. 

In a world where cloud infrastructure can be ephemeral, and where you pay for what you use when you use it - having constantly running services is a waste of money. Sure cost is important, but another very important aspect of delivering successful applications is making sure your customers have what they want when they want it. How can you accomplish both - the most efficient costs and expedient software delivery to meet customers' expectations.

## Nanoservices - Smaller, Faster, Cheaper
The concept of **nanoservices** is evolving alongside the containerized delivery of applications running on an evolving infrastructure of ephemeral, low cost cloud infrastructure. 

Kubernetes allows loose coupling at the cloud vendor level, but it isn't easy. All of the major cloud platforms provide services and features that are designed to lock you into their platform. And I get it - I work for a software company and we want to keep customers as well - so the stickier the feature, the more likely a customer is to stay with your product.

But what exactly is a nanoservice? It's just smaller than a microservice, right? Not exactly, a nanoservice is truly an architecture for Cloud Native applications and Native Kuberenetes CD allows you to easily build, test, and deploy them.

Nanoservices are Cloud Native.

Nanoservices are serverless.

Nanoservices are Functions-as-a-Service (FaaS). All the major cloud vendors have their own FaaS offerings - AWS Lambda, Azure , GCP 

One endpoint, one command.

Very low start-up time.
Very low latency between nanoservices and other services that utilize them.

The benefits of smaller, more flexible units now outweigh the costs.

"So far we have chosen not to use the serverless platforms offered by cloud providers, such as AWS’s Lambda, because they can’t guarantee such a low execution and cross-communication latency." - https://medium.com/bbc-design-engineering/powering-bbc-online-with-nanoservices-727840ba015b

## Gateway vs Mesh
The rise of microservices called for the need to communicate between them and provide the necessarry internal or external access to the services.
The containerization of microservices has made this even more complicated as containers are deployed to a cluster and there is no gurantee where they will end up.

API Gateways were first on the scene - but they had a lot of short comings when it came to intra-service communication. In most cases, a network call from one local service to another would have to go out and come back in through the API Gateway, resulting in increased latency that translated to slower response times for appliation consumers.

This becomes even more complicated in a multi-cloud or hybrid cloud deployment model. And although true multicloud is still a bit of a promise, not a reality - it is coming and with it a much more complex network topology for microservices to navigate.
Managed Istio across GCP and on-prem.

## Hybrid-Cloud or Multi-Cloud
Clear up some definitions.

Cloud agnostic, vendor lock-in, 

On-prem, in the cloud, cloud native....

## Enter Anthos
There has been a a lot of news about Anthos.

## What's Next
GKE on AWS and Azure - YES! There doesn't appear to be a timeline on this.

{{< load-photoswipe >}}