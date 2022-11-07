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

## Note

### Switching Kubernetes Context

Building the solution requires frequent changes to the current Kubernetes context, to make life simpler when doing this, you may wish to alias some of the kubectl context commands within your shell profile file:
```
gctx='kubectl config get-contexts'
uctx='kubectl config use-context'
```
Not only does kubectl use the current context but also storkctl - a Portworx command line tool that is ubiquitous with deploying Portworx PX-DR.

### Testing Load Balancer Service Endpoints

Security software from certain vendors, zscaler for example, supresses the ability to ping load balancer endpoint IP addresses, this can be worked
around using the [node-shell](https://github.com/kvaps/kubectl-node-shell) plugin for kubectl.

## Build Instructions

The instructions for building and testing the solution are as follows, click on the link
to each section for the detailed instructions pertaining to that section:
  
1. Create the AKS cluster  
   This can be carried out manually via the Azure Portal per the instructions in [this article](https://docs.portworx.com/install-portworx/cloud/azure/aks/), alternatively [this Terraform configuration](https://github.com/chrisadkin/PX-Terraform/blob/main/AKS/README.md) can be used.

2. Deploy Portworx to the AKS Cluster   
   Follow the instructions from the ["Generate Specs"](https://docs.portworx.com/install-portworx/cloud/azure/aks/) section of this article onwards in order to do this.

3. Create the GKE cluster  
   Carry this activity out via the Google Cloud Platform portal following the instructions in [this article](https://docs.portworx.com/install-portworx/cloud/gcp/gke/operator/), alternatively the cluster can be created using [this Terraform configuration](https://github.com/chrisadkin/PX-Terraform/blob/main/GKE/README.md).

4. Deploy Portworx to the GKE Cluster   
   
   **Important**
   The portworx_service service on the destination failover cluster (the GKE cluster) needs to have a ServiceType of of LoadBalancer, by
   default this is set to NodePort, to achieve this ensure that the spec contains the ```portworx.io/service-type``` annotation per this YAML
   excerpt: 
```
kind: StorageCluster
apiVersion: core.libopenstorage.org/v1
metadata:
  name: kyn-gke-destination 
  namespace: portworx
  annotations:
    portworx.io/install-source: "https://install.portworx.com/?operator=true&mc=false&b=true&kd=type%3Dpd-standard%2Csize%3D150&s=%22type%3Dpd-standard%2Csize%3D150%22&j=auto&c=px-cluster-210aba6d-2882-40f3-a50f-856e59155431&gke=true&stork=true&csi=true&mon=true&tel=false&st=k8s&promop=true"
    portworx.io/is-gke: "true"
    portworx.io/service-type: portworx-api:LoadBalancer
    .
    .
    .
```

   Follow the instructions from the ["Generate Specs"](https://docs.portworx.com/install-portworx/cloud/gcp/gke/operator/) section of this article onwards
   in order to do this.

3. Create Azure blob storage container.   

4. [Create the Portworx PX-DR cluster pair](https://docs.portworx.com/operations/operate-kubernetes/disaster-recovery/configure-migrations-to-use-service-accounts/)  

   **Important**
   
   - Cluster Pair Admin Namespace
   
   Portworx supports the concept of an "Admin namespace" - kube-system by default, if the clusterpair object is created in an admin namespace all 
   stateful application that reside on the cluster are protected by Portworx from a disaster recovery standpoint. Otherwise, if an admin namespace
   is not used, a clusterpair object needs to be created in each application namespace that requires disaster recovery.
   
   After the clusterpair is created, note that it is generated on the destination cluster (GKE) and applied to the source cluster (AKS), its health
   can be ascertained using the following command:
   ```
   storkctl get clusterpair -n kube-system
   ```
   
