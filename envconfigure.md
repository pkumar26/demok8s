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

    * *.md files - Hands-On lab instructions
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
    * azure-pipelines.yaml - Artifacts build pipeline for CD
    * appinsights-demo.yaml - Container deployment with app insights.

**Install Azure PowerShell module if it's not already installed & connect to your account - not needed if using Azure Cloud Shell**

    Install-Module -Name Az -AllowClobber
    Connect-AzAccount

**Setup deployment variables:**\
PowerShell:

    $location = 'eastus'
    $resourceGroupName = 'demok8srg'

Bash:

    location='eastus'
    resourceGroupName='demok8srg'

    # Persistence for later sessions in case of timeout - for cloud shell only
    echo location=$location >> ~/.bashrc
    echo resourceGroupName=$resourceGroupName >> ~/.bashrc


**Create resource group to hold together all deployed resources:**\
PowerShell:

    New-AzResourceGroup `
     -Name $resourceGroupName `
     -Location $location `
     -Verbose -Force

Bash:

    az group create --name $resourceGroupName --location $location
<br>
<br>
<br>
<br>

---
Continue to [Create Azure Container Registry](acr.md)\
Back to [Before you start](start.md) or to [Home](README.md)