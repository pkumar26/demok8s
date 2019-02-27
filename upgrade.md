## Upgrading and Scaling AKS cluster

**Check available upgrate for your cluster and upgrade to it**

    az aks get-upgrades --resource-group $resourceGroupName --name $aksclustername --output table
    az aks upgrade --resource-group $resourceGroupName --name $aksclustername --kubernetes-version <upgrade version>

**Wait for the cluster to be upgraded and then check the version of cluster again**

    az aks show --resource-group $resourceGroupName --name $aksclustername --output table

**Try scaling up your cluster either through portal or through command line as follows:**

    az aks scale --resource-group $resourceGroupName --name $aksclustername --node-count 5
<br>
<br>
<br>
<br>

---
Continue to [Monitoring - Logs and Application Insights](monitor.md) \
Back to [Persistent storage and state](storage.md) or to [Home](README.md)