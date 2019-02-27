## Prepare container images

>As AKS deployment is going to take a while, let's setup some images to work with.
If you already have docker images, you may push them to ACR as well.

> !! Bonus - [Create a CI pipeline for creating and publishing container images, to your container registry using Azure DevOps](https://github.com/pkumar26/azure-devops/blob/master/devops-ci.md) !!

> If not, please use sample images for demo purpose.

**Pull example images, tag them and push to your Azure Container Registry.\
You'll use these images to launch containers in AKS \
NOTE: Azure Cloud Shell cannot be used for the docker commands. Use a terminal session on your computer.**

    docker pull pkumar/helloworld:latest
    docker pull pkumar/helloworld:v2

    docker tag pkumar/helloworld <yourRegistry.azurecr.io>/helloworld
    docker tag pkumar/helloworld:v2 <yourRegistry.azurecr.io>/helloworld:v2

> 'docker image ls' should show your images

**On Azure portal find password to login to ACR. \
Your ACR registery -> Access Keys -> password**

>Note: If you are use Git Bash on Windows, you may need to prefix the `docker login` command with `winpty`.

    docker login <youracr.azurecr.io> -u <username>

    docker push <yourRegistry.azurecr.io>/helloworld
    docker push <yourRegistry.azurecr.io>/helloworld:v2
<br>
<br>
<br>
<br>

---
Continue to [Allow AKS to access ACR images](akstoacr.md) \
Back to [Create a Kubernetes cluster in Azure](aks.md) or to [Home](README.md)