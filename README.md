# Azure Kubernetes Service (AKS) with Advanced Networking

## Objectives/ Challenges

The objective of this hands-on lab is not to complete all the steps, but to understand what you are doing and take away the knowledge which could be useful in your environmet. Here's what we'll try to achieve:

- Run AKS cluster with VNet integration
- Use ARM templates for the setup
- Setup ACR & integrate it with AKS
- Push & pull container images from ACR
- Expose services on internal & public facing network
- Extend AKS cluster with ACI
- Option to setup CI & CD pipeline using Azure DevOps
- Use persistent storage and state for the DB application
- Explore Monitoring/ Application Insights

At the end of lab you'll have a vnet with 2 subnets, Azure Container Registery, AKS cluster, Public & Private LoadBalancers, 3 applications and a DB running. Here's how it'll look like:

![](diagram.jpg)

---
## Hands-on lab
[Before you start](#before-you-start) \
[Configure your environment](#configure-your-environment) \
[Setup Azure Container Registry](#setup-azure-container-registry) \
[Create a Kubernetes cluster in Azure](#create-a-kubernetes-cluster-in-azure) \
[Prepare container images](#prepare-container-images) \
[Allow AKS to access ACR images](#allow-aks-to-access-acr-images) \
[Prepare AKS cluster for HELM deployments](#prepare-aks-cluster-for-helm-deployments) \
[Private and Public Ingress](#private-and-public-ingress) \
[Deploy apps to AKS](#deploy-apps-to-aks) \
[Extend AKS](#extend-aks-with-ACI-(azure-container-images)) \
[Persistent storage and state](#persistent-storage-and-state) \
[Upgrading and Scaling AKS](#upgrading-and-scaling-aks-cluster) \
[Logging & Application Insights](#logging-&-application-insights)


## Before you start

**You may need some or all of the following:**
- [Azure account](https://azure.microsoft.com/en-us/free/)
- Code Editor e.g [VSCode](https://code.visualstudio.com/)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Docker](https://www.docker.com/get-started) - Not needed if using Azure DevOps
- [kubectl cli](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- You may choose to install - [Azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) and [PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-6) with [Azure PowerShell Module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-1.3.0) locally on your machine or use browser based [Azure Cloud Shell](https://shell.azure.com/) and select your shell there.
- [HELM](https://docs.helm.sh/)

## Configure your environment

**Login to your Azure account**

    az login
Note: Cloud Shell will automatically logs you in your account.

AKS needs permissions to manage resources in Azure & interact with Azure APIs e.g. launching LoadBalancers, virtual machines etc. Assuming that you have required permissions to create Service Principal, let's create one. \
It needs minimal permission of "Network Contributer" role on subnet in which it'll launch nodes, podes and services and of "Reader" role to pull images from ACR.

**Create Service Principal (SP):**

    az ad sp create-for-rbac -n <NameOfServicePrincipal> --skip-assignment


>You should get output similar to shown below:

    {
    "appId": "13abc4e6-2w2w-9i8u-3856-645127d63bfc",
    "displayName": "NameOfServicePrincipal",
    "name": "http://NameOfServicePrincipal",
    "password": "038855b8-35vb-44c1-1010-a3543519e9a3",
    "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
    }

>appID is your clientID \
password is your password for SP (obviously) \
and finally you need objectID

**Find Object ID of the SP created above:**

    az ad sp show --id "<appID>"

>Scroll down to the bottom of the output of above command & you'll see objectID similar to this:

    "objectId": "715d88c2-58a0-4f88-8246-7f1a304c5bed"

>Note them down as these values are needed for deploying AKS cluster.

**Clone [demok8s](https://aka.ms/aksdevops) repo locally:**

    git clone https://github.com/pkumar26/demok8s.git
    cd demok8s

>Repo has following files in it:

    * Readme.md - These instructions
    * k8sDeploy.json - Main AKS deployment template. Feel free to look but no editing is needed for this demo
    * k8sDeploy.param.json - contains values of all parameters used in main template
    * acrDeploy.json - ACR deployment template
    * acrDeploy.param.json - parameters file for ACR template
    * helloworld-internal.yaml - k8s deployment file for application to be accessible on internal network
    * helloworld-v1.yaml - k8s deployment file for Version 1 of public facing application
    * helloworld-v2.yaml - k8s deployment file for Version 2 of public facing application
    * storage.yaml - provisioning Azure disk managed storage provider
    * stateful-mongo.yaml - Mongodb with persistent storage
    * headless.yaml - Mongodb service defination
    * aci.yaml - launch containers on ACI

**Install Azure PowerShell module if it's not already installed & connect to your account - not needed if using Azure Cloud Shell**

    Install-Module -Name Az -AllowClobber
    Connect-AzAccount

**Setup deployment variables:**\
PowerShell:

    $location = 'westus'
    $resourceGroupName = 'demok8srg'

Bash:

    export location='westus'
    export resourceGroupName='demok8srg'
    # Persist for Later Sessions in Case of Timeout
    echo export location=$location >> ~/.bashrc
    echo export resourceGroupName=$resourceGroupName >> ~/.bashrc


**Create resource group to hold together all deployed resources:**\
PowerShell:

    New-AzResourceGroup `
     -Name $resourceGroupName `
     -Location $location `
     -Verbose -Force
Bash:

    az group create --name $resourceGroupName --location $location

## Create Azure Container Registry

>Explore contents of acrDeploy & acrDeploy.param.json in your favourite editor. Edit parameter values in acrDeploy.param.json as needed.

**Setup deployment variables for ACR:**\
PowerShell:

    $resourceDeploymentName = 'demoacrdeploy'
    $templatePath = $env:SystemDrive + '\' + 'users' + '\' + 'demok8s'  <-- Change it as per your environment
    $templateFile = 'acrDeploy.json'
    $templateParameterFile = 'acrDeploy.param.json'
    $template = $templatePath + '\' + $templateFile
    $templateParameter = $templatePath + '\' + $templateParameterFile

Bash:

```
resourceDeploymentName='demoacrdeploy'
#Change it as per your environment
templatePath=$HOME/demok8s
templateFile=acrDeploy.json
templateParameterFile=acrDeploy.param.json
template=$templatePath/$templateFile
templateParameter=$templatePath/$templateParameterFile
```

**Create Azure Container Registry:**\
PowerShell:

    New-AzResourceGroupDeployment `
     -Name $resourceDeploymentName `
     -ResourceGroupName $resourceGroupName `
     -TemplateFile $template `
     -TemplateParameterFile $templateParameter `
     -Verbose -Force

Bash:

```
az group deployment create \
--name $resourceDeploymentName \
--resource-group $resourceGroupName \
--template-file $template \
--parameters @$templateParameter \
--no-wait
```

## Create a Kubernetes cluster in Azure

**Generate SSH keys to be used for AKS worker nodes:**

    ssh-keygen -t rsa

**Find available AKS versions in your region:**

    az aks get-versions -l westus -o table

>Explore contents of k8sDeploy & k8sDeploy.param.json in your favourite editor. Edit parameter values in k8sDeploy.param.json as needed.

>Note:\
>Use AKS verion one older than the latest, as we'll try upgrading cluster later in the lab.\
>Use the content of .pub file generated by ssh-keygen in param.json

**Setup deployment variables for AKS:**\
PowerShell:

    $resourceDeploymentName = 'demok8sdeploy'
    $templatePath = $env:SystemDrive + '\' + 'users' + '\' + 'demok8s'  <-- Change it as per your environment
    $templateFile = 'k8sDeploy.json'
    $templateParameterFile = 'k8sDeploy.param.json'
    $template = $templatePath + '\' + $templateFile
    $templateParameter = $templatePath + '\' + $templateParameterFile

Bash:

```
resourceDeploymentName='demok8sdeploy'
# Change templatePath as per your environment
resourceDeploymentName=demok8sdeploy
templatePath=$HOME/demok8s
templateFile=k8sDeploy.json
templateParameterFile=k8sDeploy.param.json
template=$templatePath/$templateFile
templateParameter=$templatePath/$templateParameterFile
```

**Launch AKS cluster deployment:**\
PowerShell:

    New-AzResourceGroupDeployment `
     -Name $resourceDeploymentName `
     -ResourceGroupName $resourceGroupName `
     -TemplateFile $template `
     -TemplateParameterFile $templateParameter `
     -Verbose -Force

Bash:

```
az group deployment create \
--name $resourceDeploymentName \
--resource-group $resourceGroupName \
--template-file $template \
--parameters @$templateParameter \
--no-wait
```

## Prepare container images

>As AKS deployment is going to take a while, let's setup some images to work with.
If you already have docker images, you may push them to ACR as well.

> !! *Bonus - [Create a CI pipeline for creating and publishing container images, to your container registry using Azure DevOps](https://github.com/pkumar26/azure-devops/blob/master/devops-ci.md)* !!

> If not, please use sample images for demo purpose.

**Pull example images, tag them and push to your Azure Container Registry. You'll use these images to launch containers in AKS**

    docker pull pkumar/helloworld:latest
    docker pull pkumar/helloworld:v2

    docker tag pkumar/helloworld <yourRegistry.azurecr.io>/helloworld
    docker tag pkumar/helloworld:v2 <yourRegistry.azurecr.io>/helloworld:v2

> 'docker image ls' should show your images

**On Azure portal find password to login to ACR. \
Your ACR registery -> Access Keys -> password**

    docker login <youracr.azurecr.io> -u <username>

    docker push <yourRegistry.azurecr.io>/helloworld
    docker push <yourRegistry.azurecr.io>/helloworld:v2

## Allow AKS to access ACR images

**For AKS to pull images from ACR, grant AKS read only access to ACR:**\
PowerShell:

    $aksresourcegroup='demok8srg'
    $aksclustername='demok8s'
    $acrresourcegroup='demok8srg'
    $acrname=<yourRegistry.azurecr.io>

    $clientid=<yourSPclientID> or <appID> <-- created at beginning of the lab
    $acrid=$(az acr show --name $acrname --resource-group $acrresourcegroup --query "id" --output tsv)
Bash:

    export aksresourcegroup='demok8srg'
    export aksclustername='demok8s'
    export acrresourcegroup='demok8srg'
    export acrname=<yourRegistry.azurecr.io>

    export clientid=<yourSPclientID> or <appID> <-- created at beginning of the lab
    export acrid=$(az acr show --name $acrname --resource-group $acrresourcegroup --query "id" --output tsv)

>Create role assignment

    az role assignment create --assignee $clientid --role Reader --scope $acrid

>Check if AKS deployment is complete & successful. Proceed further if AKS is deployed.

**Get credentials to interact with AKS cluster:**

    az aks get-credentials --resource-group demok8srg --name demok8s

**Create K8s secret for deployments to pull images from ACR:**

    kubectl create secret docker-registry <giveSecretaName> --docker-server <yourRegistry.azurecr.io> --docker-username <clientid> --docker-password <SPPassword> --docker-email <yourEmailAddress>

>you need this (giveSecretaName) for k8s deployments.

## Prepare AKS cluster for HELM deployments

**Install Helm & verify that Tiller is up & running successfully**

    helm init
    kubectl get pods -n kube-system     <--Verify that tiller is up & running
    helm list                           <-- This should work without any error

## Private and Public Ingress

>There are multiple ways to expose services running inside AKS e.g. through Azure ALB, LB, Nginx Ingress controller, Traefik etc.
For this demo you'll be exposing internal application through Azure Internal LB & Public facing application through Traefik, which will use Azure LB.

**Install Traefik**
>If you've domain to work with, replace example domain name below. If not, you need to add example domain to hosts file as shown below:

    helm install --set dashboard.enabled=true,dashboard.domain=demok8s.example.com stable/traefik

>Wait untill LoadBalancer has IP allocated to it.

    kubectl get services

    NAME                                TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
    kubernetes                          ClusterIP      10.0.0.1      <none>        443/TCP                      4h
    virulent-dragon-traefik             LoadBalancer   10.0.75.182   <pending>     80:32426/TCP,443:30632/TCP   18s
    virulent-dragon-traefik-dashboard   ClusterIP      10.0.33.244   <none>        80/TCP                       18s
>
    kubectl get services

    NAME                                TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
    kubernetes                          ClusterIP      10.0.0.1      <none>        443/TCP                      4h
    virulent-dragon-traefik             LoadBalancer   10.0.75.182   13.14.15.16  80:32426/TCP,443:30632/TCP   18s
    virulent-dragon-traefik-dashboard   ClusterIP      10.0.33.244   <none>        80/TCP                       18s

>Note down the External-IP of Public Load Balancer.\
If you've public domain, you may setup A record to point to this IP e.g. demok8s.example.com. Accessing this will show you the dashboard of traefik. If not, please edit hosts file and add example domain pointing to LoadBalancer External IP:

    On Windows: notepad c:\windows\system32\drivers\etc\hosts
    On Linux: vim /etc/hosts

    13.14.15.16	demok8s.example.com

## Deploy apps to AKS

> !! *Bonus - [Create a CD pipeline for K8s config deployments to AKS cluster using Azure DevOps](https://github.com/pkumar26/azure-devops/blob/master/devops-cd.md)* !!
Note: This is independent of your application code which is in containers. If you decide not to use DevOps you may continue to follow the steps manually

**Deploy three applications to AKS cluster**

>Internal application, accessible only on internal network through internal load balancer


        kubectl create -f helloworld-internal.yaml
>
        kubectl get services
        NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
        helloworld-internal                 LoadBalancer   10.0.242.107   172.16.16.4     80:30304/TCP                 1m

* Note that IP of LoadBalancer is on internal subnet. You can launch a vm on this subnet (services) and access this IP in browser. Helloworld-internal is exposed through Azure internal LoadBalancer. It can also be placed behind Traefik on internal subnet and let Traefik deals with all requests. but that is out of scope of this demo.*

>Version 1 of external application, accessible through public/ external load balancer

    kubectl create -f helloworld-v1.yaml

>Version 2 of external application, accessible through public/ external load balancer

    kubectl create -f helloworld-v2.yaml

>

    kubectl get services
    NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
    helloworld-internal                 LoadBalancer   10.0.242.107   172.16.16.4     80:30304/TCP                 1m
    helloworld-v1                       NodePort       10.0.153.203   <none>          80:30303/TCP                 2m
    helloworld-v2                       NodePort       10.0.112.243   <none>          80:30304/TCP                 <invalid>
    kubernetes                          ClusterIP      10.0.0.1       <none>          443/TCP                      5h
    virulent-dragon-traefik             LoadBalancer   10.0.75.182    13.14.15.16   80:32426/TCP,443:30632/TCP   1h
    virulent-dragon-traefik-dashboard   ClusterIP      10.0.33.244    <none>          80/TCP                       1h

>Note that helloworld-v1 & v2 don't have external IPs but they are accessible through Traefik

http://demo.example.com/v1 \
http://demo.example.com/v2

**Scale up the any application by editing yaml file and changing replicas from 1 to 3**

    replicas: 3

**Apply changes, and verify the increased number of pods**

    kubectl apply -f helloworld-v1.yml
    kubectl get pods

**Kill some pods & see what happens, e.g**

    kubectl delete pod helloworld-v1-deployment-567bf9876-2r2km
    kubectl get pods

## Extend AKS with ACI (Azure Container Images)

> !! *Bonus - [Extend AKS cluster workloads to Azure Container Instances (ACI), without adding any nodes to your cluster](aci.md)* !!

## Persistent storage and state

**Deploy Mongodb cluster with persistent storage as a stateful set**

    kubectl apply -f storage.yaml
    kubectl apply -f headless.yaml
    kubectl apply -f stateful-mongo.yaml
>
    kubectl get pods -o wide
    NAME                                             READY     STATUS    RESTARTS   AGE       IP             NODE
    cantankerous-blackbird-traefik-54ffb6ddc-ctxlp   1/1       Running   3          85d       172.16.0.26    aks-agentpool-23040724-2
    mongo-0                                          2/2       Running   0          29m       172.16.0.76    aks-agentpool-23040724-0
    mongo-1                                          2/2       Running   0          27m       172.16.0.20    aks-agentpool-23040724-2
    mongo-2                                          2/2       Running   0          25m       172.16.0.123   aks-agentpool-23040724-1

**Login to mongo-0 and check if it is the primary node**

    kubectl exec -it mongo-1 bash

    root@mongo-0:/# mongo
    MongoDB shell version v4.0.6
    .....
    rs0:PRIMARY>

**Kill primary container node mongo-0 and observe what happens**

    kubectl delete pod mongo-0
    kubectl get pods

## Upgrading and Scaling AKS cluster

**Check available upgrate for your cluster and upgrade to it**

    az aks get-upgrades --resource-group demok8srg --name demok8s --output table
    az aks upgrade --resource-group demok8srg --name demok8s --kubernetes-version <upgrade version>

**Wait for the cluster to be upgraded and then check the version of cluster again**

    az aks show --resource-group demok8srg --name demok8s --output table

**Try scaling up your cluster either through portal or through command line as follows:**

    az aks scale --name demok8s --resource-group demok8srg --node-count 5


## Logging & Application Insights

Browse to your AKS cluster on the Azure portal and click on Insights under Monitoring. Check cluster, nodes or container info. Click on the Containers tab and pick a container to view its live logs and debug what is going on...

![](logging/1-monitoring.jpg)

![](logging/2-monitoring.jpg)

![](logging/3-monitoring.jpg)

**Enable APM to get insights into your application**

This concludes the demo.
