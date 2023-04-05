---
title: Cross Region Disaster Recovery for Jenkins on Kubernetes
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","DR"]
part: 4
date: 2021-05-09T08:49:15-04:00
draft: true
---
I am a big fan of running Jenkins on Kubernetes and one of the many reasons is because of the portability offered by Kubernetes. However, that portability becomes more challenging when running stateful applications on Kubernetes.

Jenkins is a stateful application and uses file based storage for its configuration and logs.

# Disaster Recovery is Not High Availability

Although both High Availability (HA) and Disaster Recovery (DR) main purpose is to ensure that a service is available. 

## Multi-Zone vs Multi-Region

Most, if not all, major cloud vendors regions have at least 3 availability zones in them. So it is very unlikely that an entire region will become unavailable. But it has happened and from a software services perspective, that is a disaster.

Jenkins is typically run as a statefulset on Kubernetes and is typically tied to one availability zone due to its dependency on persistent storage. 

Using persistent block storage disk for performance. 

GKE offers the use of high performing persistent disk across two zones.

For EKS and AKS you would need to use networks storage, such as EFS. More expensive, worse performance but available across all the zones in a region.

## The Container Storage Interface and Kubernetes Volume Snapshots

The Container Storage Interface (CSI) is quickly become the standard interface for Kubernetes storage. 

It is automatically installed with AKS and GKE v1.19, and the previous in-tree storage drivers are deprecated.

The EBS CSI Driver only became Generally Available in late April of 2021 - even though it has been a GA feature of Kubernetes since v1.13.

## Velero ...and other Tools

Velero 1.4 added beta support for CSI and, more specifically, for native Kubernetes VolumeSnapshots. Before adding support for CSI snapshots, Velero relied on its own custom snapshot capability or the use of Restic. 

## Data Source for Persistent Volume Claims

The PersistentVolumeClaim.DataSource parameter was added with Kubernetes 1.12. 

Two types of data sources: VolumeSnapshot and PersistentVolumeClaim.