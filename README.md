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
    - roles/iam.serviceAccountUser
    - roles/compute.admin
    - roles/container.clusterViewer    
  - Cluster OATH scopes: 
    - https://www.googleapis.com/auth/logging.write
    - https://www.googleapis.com/auth/cloud-platform
    - https://www.googleapis.com/auth/monitoring
    
- AKS Cluster
  - Three worker node cluster
  - Worker node machine type: standard_d8as_v4
  - Worker node image type: Linux
  - Boot disk size per node: 128 GB
  - Kubernetes version: 1.23.8
  - A service principal with the following custom role definition:

```
az role definition create --role-definition '{
"Name": "<your-role-name>",
"Description": "",
"AssignableScopes": [
    "/subscriptions/<your-subscription-id>"
],
"Actions": [
    "Microsoft.ContainerService/managedClusters/agentPools/read",
    "Microsoft.Compute/disks/delete",
    "Microsoft.Compute/disks/write",
    "Microsoft.Compute/disks/read",
    "Microsoft.Compute/virtualMachines/write",
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachineScaleSets/virtualMachines/write",
    "Microsoft.Compute/virtualMachineScaleSets/virtualMachines/read"
],
"NotActions": [],
"DataActions": [],
"NotDataActions": []
}'
```

  - Load balancer sku: standard
  
- Object storage for PX-DR: Azure blob storage

- Portworx:
  - Portworx Enterprise 2.10
  - PX-DR
  
## Build Instructions

The instructions for building and testing the solution are as follows, click on the link
to each section for the detailed instructions pertaining to that section:
  
1. Create the AKS cluster  
   This can be carried out manually via the Azure Portal per the instructions in [this article](https://docs.portworx.com/install-portworx/cloud/azure/aks/), alternatively [this Terraform configuration](https://github.com/chrisadkin/PX-Terraform/blob/main/AKS/README.md) can be used

2. Deploy Portworx to the AKS Cluster   
   Follow the instructions from the ["Generate Specs"](https://docs.portworx.com/install-portworx/cloud/azure/aks/) section of this article onwards in order to do this.

3. Create the GKE cluster  
   Carry this activity out via the Google Cloud Platform portal following the instructions in [this article](https://docs.portworx.com/install-portworx/cloud/gcp/gke/operator/), alternatively the cluster can be created using [this Terraform configuration](https://github.com/chrisadkin/PX-Terraform/blob/main/GKE/README.md).

4. Deploy Portworx to the GKE Cluster   
   Follow the instructions from the ["Generate Specs"](https://docs.portworx.com/install-portworx/cloud/gcp/gke/operator/) section of this article onwards in order to do this.

3. Create Azure blob storage container   
4. [Deploy Portworx Enterprise to each cluster](https://github.com/chrisadkin/PX-DR-AKS-To-GKE/blob/main/deploy-portworx-enterprise/README.md)
5. [Configure PX-DR](https://github.com/chrisadkin/PX-DR-AKS-To-GKE/blob/main/configure-px-dr/README.md)
6. [Test failover](https://github.com/chrisadkin/PX-DR-AKS-To-GKE/blob/main/test-failover/README.md)
