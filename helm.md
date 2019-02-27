## Prepare AKS cluster for HELM deployments

**Install Helm & verify that Tiller is up & running successfully**

    helm init
    # Verify that tiller is up & running
    kubectl get pods -n kube-system
    # This should work without any error
    helm list
<br>
<br>
<br>
<br>


Continue to [Private and Public Ingress](ingress.md) \
Back to [Allow AKS to access ACR images](akstoacr.md) or to [Home](README.md)