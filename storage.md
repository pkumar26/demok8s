## Persistent storage and state

**Deploy Mongodb cluster with persistent storage as a stateful set**

    kubectl apply -f storage.yaml
    kubectl apply -f headless.yaml
    kubectl apply -f stateful-mongo.yaml
> These deployments may take several minutes to complete. Use this time to review the manifests we just applied.

    kubectl get pods -o wide --watch
    NAME                                             READY     STATUS    RESTARTS   AGE       IP             NODE
    cantankerous-blackbird-traefik-54ffb6ddc-ctxlp   1/1       Running   3          85d       172.16.0.26    aks-agentpool-23040724-2
    mongo-0                                          2/2       Running   0          29m       172.16.0.76    aks-agentpool-23040724-0
    mongo-1                                          2/2       Running   0          27m       172.16.0.20    aks-agentpool-23040724-2
    mongo-2                                          2/2       Running   0          25m       172.16.0.123   aks-agentpool-23040724-1

**Login to mongo-0 and check if it is the primary node**

    kubectl exec -it mongo-0 bash

    root@mongo-0:/# mongo
    MongoDB shell version v4.0.6
    .....
    rs0:PRIMARY>

**Kill primary container node mongo-0 and observe what happens**

    kubectl delete pod mongo-0
    kubectl get pods

Connection string for the applications to connect to mongodb in this example will appear like this:

    mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo:27017/contentdb?replicaSet=rs0
<br>
<br>
<br>
<br>

---
Continue to [Upgrading and Scaling AKS cluster](upgrade.md) \
Back to [Extend AKS](aci.md) or to [Home](README.md)