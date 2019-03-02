## Create Azure KeyVault

Create Azure KeyVault using [ARM](https://github.com/pkumar26/keyvault) or using [Azure cli](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-manage-with-cli2) and then come back to this page for its integration with AKS:

**Add secrets to the KeyVault:**

PowerShell:

    $mysharedkv = 'name_of_keyvault'

    $secretvalue = ConvertTo-SecureString 'ssh_key_here' -AsPlainText -Force
    $secret = Set-AzKeyVaultSecret -VaultName $mysharedkv -Name 'custom_secret_name' -SecretValue $secretvalue

    $secretvalue = ConvertTo-SecureString 'service_principal_appid_here' -AsPlainText -Force
    $secret = Set-AzKeyVaultSecret -VaultName $mysharedkv -Name 'custom_secret_name' -SecretValue $secretvalue

    $secretvalue = ConvertTo-SecureString 'service_principal_objectid_here' -AsPlainText -Force
    $secret = Set-AzKeyVaultSecret -VaultName $mysharedkv -Name 'custom_secret_name' -SecretValue $secretvalue

    $secretvalue = ConvertTo-SecureString 'service_principal_secret_here' -AsPlainText -Force
    $secret = Set-AzKeyVaultSecret -VaultName $mysharedkv -Name 'custom_secret_name' -SecretValue $secretvalue

Bash:

    mysharedkv='name_of_keyvault'

    az keyvault secret set --vault-name $mysharedkv --name 'custom_secret_name' --value 'ssh_key_here'
    az keyvault secret set --vault-name $mysharedkv --name 'custom_secret_name' --value 'service_principal_appid_here'
    az keyvault secret set --vault-name $mysharedkv --name 'custom_secret_name' --value 'service_principal_objectid_here'
    az keyvault secret set --vault-name $mysharedkv --name 'custom_secret_name' --value 'service_principal_secret_here'


**Try retrieving secrets from the KeyVault:**

PowerShell:

    (Get-AzKeyVaultSecret -vaultName $mysharedkv -name "custom_secret_name").SecretValueText

Bash:

    az keyvault secret show --vault-name $mysharedkv --name 'custom_secret_name' --query 'value'

**Modify k8sDeploy.json and k8sDeploy.param.json**

- k8sDeploy.json: For all the parameters whose values are stored in keyvault, change their "type" from "string" to "securestring" as shown below. \
In this example, change values of SPAppID, SPObjectID, SPSecret & sshPubKey

        "SPAppID": {
            "type": "securestring",
            "metadata": {
                "description": "APP ID of Service Principal"
            }
        }

- Find id of your keyvault

  PowerShell:

        Get-AzKeyVault

    Shell:

        az keyvault list

- k8sDeploy.param.json: Update the values of above mentioned parameters as shown below.

        "SPAppID": {
            "reference": {
                "keyVault": {
                    "id": "id_of_your_keyvault"
                },
                "secretName": "custom_secret_name"
            }
        }

**Save modified files and [Create AKS cluster in Azure](aks.md)**
<br>
<br>
<br>
<br>

---
Back to [Create a Kubernetes cluster in Azure](aks.md) or to [Home](README.md)


