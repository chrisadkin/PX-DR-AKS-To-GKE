# PX-DR-AKS-To-GKE

## Overview

A repository to provide instructions on setting up PX-DR with asynchronous replication between AKS and GKE resulting in a solution represented by the
diagram below:
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/chrisadkin/PX-DR-AKS-To-GKE/blob/main/images/HLA.png?raw=true">

## Solution Bill of Materials

- GKE Cluster
  - Three worker node regional cluster, zones: europe-west2-a, europe-west2-b and europe-west2-c
  - Worker node machine type: e2-standard-8 
  - Worker node image type: Ubuntu with containerd
  - Boot disk size per node: 100 GB 
  - Kubernetes version 1.22.12-gke.2300 
  - Cluster service account roles:
  - Cluster OATH scopes: 

- AKS Cluster
  - Three worker node cluster
  - Worker node machine type: standard_d8as_v4
  - Worker node image type: Linux
  - Boot disk size per node: 128 GB
  - Kubernetes version: 1.23.8
  - Cluster service principal roles:
  - Load balancer sku: standard
  
- Object storage for PX-DR: Azure blob storage

- Portworx:
  - Portworx Enterprise 2.10
  - PX-DR
  
## Build Instructions

The instructions for building and testing the solution are as follows, click on the link
to each section for the detailed instructions pertaining to that section:

1. Create AKS and GKE clusters
2. Create Azure blob storage container
3. [Deploy Portworx Enterprise to each cluster]()
4. [Configure PX-DR]()
5. [Test failover]()
