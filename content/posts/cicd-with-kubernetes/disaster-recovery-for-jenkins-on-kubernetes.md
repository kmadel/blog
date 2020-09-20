---
title: Cross Region Disaster Recovery for Jenkins on Kubernetes
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","DR"]
part: 4
date: 2020-08-09T08:49:15-04:00
draft: true
---
I am a big fan of running Jenkins on Kubernetes and one of the main reasons is because of the portability offered by Kubernetes.

Jenkins is a stateful application and uses file based storage for its configuration and logs.

# Failover, HA and Disaster Recovery

## Multi-Zone vs Multi-Region

## The Container Storage Interface and Kubernetes Volume Snapshots

## Velero ...and other Tools

Velero 1.4 added beta support for CSI and, more specifically, for native Kubernetes VolumeSnapshots.

## Data Source for Persistent Volume Claims

The PersistentVolumeClaim.DataSource parameter was added with Kubernetes 1.12. 

Two types of data sources: VolumeSnapshot and PersistentVolumeClaim.