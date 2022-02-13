---
title: Minimalistic guide to Launch Azure Red Hat Openshift
description: Simplified notes to launch Azure Red Hat Openshift Cluster
tags: 'azure,redhat,openshift,aro'
canonical_url: null
published: true
---
![](./assets/aro-1.jpeg)

## What is ARO
Azure Red Hat OpenShift (ARO) is a fully-managed service of Red Hat OpenShift on Azure, Jointly engineered, managed, and supported by Microsoft and Red Hat. Azure Red Hat OpenShift takes care of management, freeing developers to focus on developing new services. Azure Red Hat OpenShift offers a highly-available control plane with fully-managed master and application nodes — no virtual machines to operate, no patching required, backed by the expertise of global Site Reliability Engineers.

This blog post is a simplified version of the [official documentation](https://docs.microsoft.com/en-us/azure/openshift/tutorial-create-cluster), Read On.

![Azure Red Hat OpenShift](https://cloud.redhat.com/hubfs/images/illustrations/OSH-ContainerStack-MarketectureDesktop-2x.png)

## Prerequisites

- Azure account and portal access
- ARO requires `roleAssignment/write` permission, make sure your Azure User account has
`Microsoft.Authorization/roleAssignments/write` permissions, such as `User Access Administrator` or `Owner` [more info here](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)

![](./assets/aro-2.png)

- ARO requires a minimum of 40 cores to create and run an OpenShift cluster. The default Azure resource quota for a new Azure subscription is 10 and does not meet this requirement. Increase quota from 10 to minimum 40. To request an increase in your resource limit, see [Standard quota: Increase limits by VM series](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/per-vm-quota-requests)

- Launch Azure Cloud Shell from Azure Portal (top right). We would be using Azure Cloud Shell for this tutorial.
- Export some variables that we will often use in the rest of the tutorial.
```
export LOCATION=centralindia
export RESOURCEGROUP=ksingh-resource-group-india
export CLUSTER=azureopenstack
```
- Verify the quota
```
az vm list-usage -l $LOCATION \
--query "[?contains(name.value, 'standardDSv3Family')]" \
-o table
```
-  Grab subscription ID from Azure Portal
```
az account set --subscription <SUBSCRIPTION ID>
```
- Grab OpenShift Pull secrets for your ARO cluster from (cloud.redhat.com)[https://cloud.redhat.com/openshift/install/azure/aro-provisioned]
- Download the pull-secret file and store that on Azure cloud shell as `pull-secret.txt`

## Launching ARO Cluster

- Register the resource providers
```
az provider register -n Microsoft.RedHatOpenShift --wait
az provider register -n Microsoft.Compute --wait
az provider register -n Microsoft.Storage --wait
az provider register -n Microsoft.Authorization --wait
```
- Create a resource group
```
az group create --name $RESOURCEGROUP --location $LOCATION
```
- Create a virtual network
```
az network vnet create --resource-group $RESOURCEGROUP --name aro-vnet --address-prefixes 10.0.0.0/22
```
- Create two subnets in `aro-vnet` network for OpenShift control plane (master) and worker nodes
```
az network vnet subnet create --resource-group $RESOURCEGROUP --vnet-name aro-vnet --name master-subnet --address-prefixes 10.0.0.0/23 --service-endpoints Microsoft.ContainerRegistry

az network vnet subnet create --resource-group $RESOURCEGROUP --vnet-name aro-vnet --name worker-subnet --address-prefixes 10.0.2.0/23 --service-endpoints Microsoft.ContainerRegistry
```
- Update master node subnet network policy
```
az network vnet subnet update --name master-subnet --resource-group $RESOURCEGROUP --vnet-name aro-vnet --disable-private-link-service-network-policies true
```
- Finally, create ARO cluster with default configuration
```
az aro create --resource-group $RESOURCEGROUP --name $CLUSTER --vnet aro-vnet --master-subnet master-subnet --worker-subnet worker-subnet --pull-secret @pull-secret.txt
```

## Connect to ARO
- (GUI) Grab OpenShift Console URL and credentials
```
az aro show --name $CLUSTER --resource-group $RESOURCEGROUP --query "consoleProfile.url" -o tsv
az aro list-credentials --name $CLUSTER --resource-group $RESOURCEGROUP
```
- (CLI) Install OpenShift Client `oc`
```
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz

mkdir openshift
tar -zxvf openshift-client-linux.tar.gz -C openshift
echo 'export PATH=$PATH:~/openshift' >> ~/.bashrc && source ~/.bashrc
```
- Connecting to OpenShift CLI
```
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```
## Summary
The experience of launching OpenShift cluster from Azure Cloud Shell `aro` is very simple and easy. 
Hope this helps, See You Next Time!