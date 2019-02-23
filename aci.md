**Create an additional subnet for ACI**

    az network vnet subnet create
        --resource-group demok8srg
        --vnet-name demok8svnet
        --name aci-subnet
        --address-prefix 172.16.32.0/28

**Add Azure CLI extension**

    az aks install-connector --resource-group demok8srg --name demok8s --connector-name virtual-kubelet --os-type Both

**Verify the node is available**

    kubectl get node
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-agentpool-23040724-0   Ready     agent     85d       v1.9.11
    aks-agentpool-23040724-1   Ready     agent     85d       v1.9.11
    aks-agentpool-23040724-2   Ready     agent     85d       v1.9.11
    virtual-kubelet-virtual-kubelet-linux-eastus     Ready     agent     33m       v1.13.1-vk-v0.7.4-44-g4f3bd20e-dev
    virtual-kubelet-virtual-kubelet-windows-eastus   Ready     agent     33m       v1.13.1-vk-v0.7.4-44-g4f3bd20e-dev

**Examine aci.yaml especially nodeSelector & tolerations sections. Redeploy helloworld-v2 app on ACI**

    kubectl delete -f helloworld-v2.yaml
    kubectl create -f aci.yaml
Browse
    http://demok8s.example.com/v2