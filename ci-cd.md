# Continous Integration & Continous Delivery with Azure DevOps

*You can also use Azure DevOps as follows to create your images & push them to your Azure container registry*

    Go to https://dev.azure.com and sign-in with your Azure subscription credentials.
    If this is your first time to provision an Azure DevOps account, you’ll be taken through a quick wizard to create a new organization.

    Create a new private project, and give it a name.
    Click on Repos then import the code of the captureorder service from the public GitHub repository located at https://github.com/pkumar26/containers.git

    Build pipeline for the application Docker container is included in the repo.
    Build the code in azch-captureorder as a Docker image and push it to the Azure Container Registry you provisioned before.
    Choose YAML as the pipeline template
    Browse to and select the azure-pipelines.yml included in the repo. You may also change the agent to be Hosted Ubuntu
    Define variables in your build pipeline in the web UI:
        dockerId: The admin user name/Service Principal ID for the Azure Container Registry.
        acrName: The Azure Container Registry name.
        dockerPassword: The admin password/Service Principal password for Azure Container Registry.
    Run the build pipeline and verify that it works
