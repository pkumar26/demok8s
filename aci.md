**Create an additional subnet for ACI**

    az network vnet subnet create \
        --resource-group demok8srg \
        --vnet-name demok8svnet \
        --name aci-subnet \
        --address-prefix 10.16.34.0/28

**Add Azure CLI extension**

    az extension add --source https://aksvnodeextension.blob.core.windows.net/aks-virtual-node/aks_virtual_node-0.2.0-py2.py3-none-any.whl

**Enable the virtual node in your cluster**

    az aks enable-addons \
        --resource-group akschallenge \
        --name <your-aks-cluster-name> \
        --addons virtual-node \
        --subnet-name aci-subnet

**Verify the node is available**

    kubectl get node
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-agentpool-23040724-0   Ready     agent     85d       v1.9.11
    aks-agentpool-23040724-1   Ready     agent     85d       v1.9.11
    aks-agentpool-23040724-2   Ready     agent     85d       v1.9.11
    virtual-node-aci-linux     Ready     agent     48m       v1.13.1-vk-v0.7.4-44-g4f3bd20e-dev

**Examine aci.yaml and redeploy internal app on ACI**

    kubectl delete -f helloworld-internal.yaml
    kubectl create -f aci.yaml