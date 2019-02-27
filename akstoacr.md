## Allow AKS to access ACR images

**For AKS to pull images from ACR, grant AKS read only access to ACR:**\
PowerShell:

    $aksresourcegroup='demok8srg'
    $aksclustername='demok8s'
    $acrresourcegroup='demok8srg'
    $acrname='<yourRegistry>'

    $clientid=<yourSPclientID> or <appID> <-- created at beginning of the lab
    $acrid=$(az acr show --name $acrname --resource-group $acrresourcegroup --query "id" --output tsv)
Bash:

    aksresourcegroup='demok8srg'
    aksclustername='demok8s'
    acrresourcegroup='demok8srg'
    acrname='<yourRegistry>'
    # Persist for Later Sessions in Case of Timeout
    echo export aksclustername=$aksclustername >> ~/.bashrc
    echo export acrname=$acrname >> ~/.bashrc

    clientid=<yourSPclientID> or <appID> <-- created at beginning of the lab
    acrid=$(az acr show --name $acrname --resource-group $acrresourcegroup --query "id" --output tsv)

>Create role assignment

    az role assignment create --assignee $clientid --role Reader --scope $acrid

>Check if AKS deployment is complete & successful. Proceed further if AKS is deployed.

**Get credentials to interact with AKS cluster:**

    az aks get-credentials --resource-group $resourceGroupName --name $aksclustername

**Create K8s secret for deployments to pull images from ACR:**

    kubectl create secret docker-registry <giveSecretaName> --docker-server <yourRegistry.azurecr.io> --docker-username <clientid> --docker-password <SPPassword> --docker-email <yourEmailAddress>

>You need this (giveSecretaName) for k8s deployments.

<br>
<br>
<br>
<br>

---
Continue to [Prepare AKS cluster for HELM deployments](helm.md) \
Back to [Prepare container images](prepareimages.md) or to [Home](README.md)