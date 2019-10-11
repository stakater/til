# Managed Openshift On Azure 

To deploy `Red Hat OpenShift Container Platform Self-Managed` on Azure, you first need your private key in the Azure Key Vault, so you can refer that when creating OCP.

## Azure Key Vault

Run following commands to create a secret in Azure Key Vault with your private key.

```sh
az group create --name keyvaultrg --location westeurope   # Create resource group
az keyvault create --resource-group keyvaultrg --name ocp-keyvault --enabled-for-template-deployment true --location westeurope      # Create Key Vault
ssh-keygen -f ~/.ssh/openshift_rsa -t rsa -N ''   # Generate ssh key, can use existing as well
az keyvault secret set --vault-name ocp-keyvault --name keysecret --file ~/.ssh/openshift_rsa  # Set openshift_rsa file private key in the secret
```

## Create Red Hat OpenShift Container Platform Self-Managed

Go to Azure portal, and create a new resource, select `Red Hat OpenShift Container Platform Self-Managed`, It will ask for parameters, fill them, with their values.

Run `cat ~/.ssh/openshift_rsa.pub` and copy that, it will be used in the paramaters.

Check [Azure Docs](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/openshift-marketplace-self-managed) for help in filling the values.

