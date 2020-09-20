---
title: Mapping IAM and Kubernetes Service Accounts
series: ["CICD on Kubernetes"]
tags: ["Kubernetes","Jenkins","CI","CD","IAM"]
part: 5
date: 2020-09-25T12:49:15-04:00
draft: true
---


Links
"Workload Identity uses updated Kubernetes authentication tokens, available in beta since Kubernetes 1.12, to more effectively and efficiently authenticate Kubernetes-based services as they interact with other Google Cloud Platform (GCP) services."
https://searchitoperations.techtarget.com/news/252467405/Kubernetes-authentication-project-wrestles-with-migration-problems

https://github.com/kubernetes/community/blob/master/contributors/design-proposals/auth/bound-service-account-tokens.md

https://github.com/imduffy15/k8s-gke-service-account-assigner

https://github.com/jtblin/kube2iam

The Google Kubernetes Method:

The recommended technique for Kubernetes is to create a separate service account for each application that runs in the cluster and reduce the scopes applied to the default service account. The roles assigned to each service account vary based upon the permissions that the applications require.

Service account credentials (key) are downloaded as a Json file and then stored in Kubernetes as a Secret. You would then mount the volume with the secret (the IAM Service Acount key file). The application running in the container will then need to have that Kubernetes secret mounted and then use the key file to authenticate before using Google Cloud services.

This command will store the downloaded credentials file into a Kubernetes secret volume as the secret named service-account-credentials. The credentials file inside Kubernetes is named key.json. The credentials are loaded from the file that was downloaded from Google Cloud named `/secrets/credentials.json





