## Deploy apps to AKS

> !! Bonus - [Create a CD pipeline for K8s config deployments to AKS cluster using Azure DevOps](https://github.com/pkumar26/azure-devops/blob/master/devops-cd.md) !!
Note: This is independent of your application code which is in containers. If you decide not to use DevOps you may continue to follow the steps manually

**Deploy three applications to AKS cluster**

>Internal application, accessible only on internal network through internal load balancer.

>In your preferred code editor, edit kubernetes deployment files like helloworld-internal.yaml, helloworld-v1.yaml, helloworld-v2.yaml etc.\
Replace <yourACRRegistry.azurecr.io> with the name of you ACR registry.\
Replace \<k8sSecretName> with the secret you created earlier.


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

http://demok8s.example.com/v1 \
http://demok8s.example.com/v2

**Scale up the any application by editing helloworld-v1.yaml file and changing replicas from 1 to 3**

    replicas: 3

**Apply changes, and verify the increased number of pods**

    kubectl apply -f helloworld-v1.yaml
    kubectl get pods

**Try temporarily changing number of replicas by editing deployment defination on the fly:**

    kubectl get deployments
    kubectl edit <your_deployment_name>

**Kill some pods & see what happens, e.g**

    kubectl delete pod helloworld-v1-deployment-567bf9876-2r2km
    kubectl get pods
<br>
<br>
<br>
<br>

---
Continue to [Extend AKS with ACI](aci.md) \
Back to [Private and Public Ingress](ingress.md) or to [Home](README.md)