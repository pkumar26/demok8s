# K8s Hands-On Lab's Objectives:
- Running AKS cluster with VNet integration
- ARM templates for the setup
- ACR setup & its integration with AKS
- Push & pull container images from ACR
- Expose two services: one on internal & another on external network
- Use HELM for deployments to AKS (Optional)



## Before you start
- Azure account
- Code Editor e.g VSCode - I think it's the best
- docker cli
- kubectl cli
- PowerShell or PowerShell ISE
- az cli
- HELM (optional) - Only if you decide to use HELM
- Draft (optional)

# Prepare

*Login to your Azure account*
    
    az login

AKS needs permissions to manage resources in Azure & interact with Azure APIs e.g. launching LoadBalancers, virtual machines etc. Assuming that you have required permissions to create Service Principal, let's create one. \
It needs minimal permission of "Network Contributer" role on subnet in which it'll launch nodes, podes and services and of "Reader" role to pull images from ACR.

*Create Service Principal (SP):*

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

*Find Object ID of the SP created above:*
    
    az ad sp show --id "<appID>"

>If you scroll down to the bottom of the output of above command, you'll see objectID similar to this:

    "objectId": "715d88c2-58a0-4f88-8246-7f1a304c5bed"

>At this point, you are ready to launch AKS cluster. 

*Clone [demok8s](https://github.com/pkumar26/demok8s.git) repo locally:*

    git clone https://github.com/pkumar26/demok8s.git

>Repo has following files in it:

    * Readme.md - These instructions
    * k8sDeploy.json - Main AKS deployment template. Feel free to look but no editing is needed for this demo.
    * k8sDeploy.param.json - contains values of all parameters used in main template.
    * acrDeploy.json - ACR deployment template.
    * acrDeploy.param.json - parameters file for ACR template

*Open Powershell/ PowerShell ISE and setup deployment variables:*

    $location = 'westus2'
    $resourceGroupName = 'demok8srg'

*Create resource group if it not already there:*
    
    New-AzureRmResourceGroup `
     -Name $resourceGroupName `
     -Location $location `
     -Verbose -Force

>Open acrDeploy.param.json in your favourite editor. Go through each value and edit them as needed

*Setup deployment variables for ACR:*

    $location = 'westus2'
    $resourceGroupName = 'demok8srg'
    $resourceDeploymentName = 'demoacrdeploy'
    $templatePath = $env:SystemDrive + '\' + 'users' + '\' + 'demo'  <-- Change it as per your environment
    $templateFile = 'acrDeploy.json'
    $templateParameterFile = 'acrDeploy.param.json'
    $template = $templatePath + '\' + $templateFile
    $templateParameter = $templatePath + '\' + $templateParameterFile

*Create Azure Container Registry:*

    New-AzureRmResourceGroupDeployment `
     -Name $resourceDeploymentName `
     -ResourceGroupName $resourceGroupName `
     -TemplateFile $template `
     -TemplateParameterFile $templateParameter `
     -Verbose -Force

>Open k8sDeploy.param.json in your favourite editor. Go through each value and edit them as needed

*Setup deployment variables for AKS:*

    $location = 'westus2'
    $resourceGroupName = 'demok8srg'
    $resourceDeploymentName = 'demok8sdeploy'
    $templatePath = $env:SystemDrive + '\' + 'users' + '\' + 'demo'  <-- Change it as per your environment
    $templateFile = 'k8sDeploy.json'
    $templateParameterFile = 'k8sDeploy.param.json'
    $template = $templatePath + '\' + $templateFile
    $templateParameter = $templatePath + '\' + $templateParameterFile

*Launch AKS cluster deployment:*

    New-AzureRmResourceGroupDeployment `
     -Name $resourceDeploymentName `
     -ResourceGroupName $resourceGroupName `
     -TemplateFile $template `
     -TemplateParameterFile $templateParameter `
     -Verbose -Force

>As AKS deployment is going to take a while, let's setup some images to work with.
If you already have docker images to work with, you may push them to ACR as well. If not, please use sample images for demo purpose.


*Pull example images, tag them and push to your Azure Container Registry. We'll use these images to launch containers in AKS*

    docker pull pkumar/helloworld:latest
    docker pull pkumar/helloworld:v2

    docker tag pkumar/helloworld <yourRegistry.azurecr.io>/helloworld
    docker tag pkumar/helloworld:v2 <yourRegistry.azurecr.io>/helloworld:v2

> 'docker image ls' should show your images

*On Azure portal find password to login to ACR. \
Your ACR registery -> Access Keys -> password*

    docker login youracr.azurecr.io -u <username>

    docker push <yourRegistry.azurecr.io>/helloworld
    docker push <yourRegistry.azurecr.io>/helloworld:v2



*For AKS to pull images from ACR, grant AKS read only access to ACR:*

    $aksresourcegroup='demok8srg'
    $aksclustername='demok8s'
    $acrresourcegroup='sharedrg'
    $acrname=<yourRegistry.azurecr.io>
    $clientid=<yourSPclientID> or <appID> <-- created at beginning of the lab 

    $acrid=$(az acr show --name $acrname --resource-group $acrresourcegroup --query "id" --output tsv)

    az role assignment create --assignee $clientid --role Reader --scope $acrid

>Check if AKS deployment is complete & successful. Proceed further if AKS is deployed.

*Get credentials to interact with AKS cluster:*

    az aks get-credentials --resource-group demok8srg --name demok8s

*Create K8s secret for deployments to pull images from ACR:*

    kubectl create secret docker-registry <giveSecretaName> --docker-server <yourRegistry.azurecr.io> --docker-username <clientid> --docker-password <SPPassword> --docker-email <yourEmailAddress>

*Install Helm & verify that Tiller is up & running successfully**

    helm init
    helm list   ….. This should work without any error

    kubectl get pods -n kube-system


*Install Traefik*
>If you've domain to work with:

    helm install --set dashboard.enabled=true,dashboard.domain=<URLto accessdashboard> stable/traefik
>otherwise:

    helm install stable/traefik
Updating ....