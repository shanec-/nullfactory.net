---
layout: post
title: Deploying a Azure Classic Storage Account using Azure Resource Manager
category: Azure, Deployment, Azure Storage Accounts
---
I've been working on Azure Resource Group templates quite a bit over the last few weeks. While it has been a pleasant experience overall, I ran into some hurdles the other day while attempting to figure out how to create a `Microsoft.ClassicStorage/StorageAccounts` using the Azure Resource Manager(ARM) API.

The latest version (2.7 at the time of writing) of Azure SDK GUI tools for visual studio were not particularly helpful in generating the required json, but thankfully [this](http://stackoverflow.com/questions/27347200/add-storage-to-azure-resource-manager) post pointed me in the right direction. And after a little bit of fiddling I find that `2015-06-01` appears to be last supported `apiVersion` that works with classic storage.

 [![Azure PowerShell Unsupported](/images/posts/DeployClassicStorageArm/10_SupportedVersion.png)](/images/posts/DeployClassicStorageArm/10_SupportedVersion.png)

Here's the final script I used to create a classic storage container: 

    {
	    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
	    "contentVersion": "1.0.0.0",
	    "parameters": {
		    "PrimaryStorageName": {
		    	"type": "string"
		    },
		    "PrimaryStorageType": {
			    "type": "string",
			    "defaultValue": "Standard_LRS",
			    "allowedValues": [
				    "Standard_LRS",
				    "Standard_GRS",
				    "Standard_ZRS"
			    ]
		    },
		    "PrimaryStorageLocation": {
		    "type": "string",
		    "defaultValue": "East US",
		    "allowedValues": [
			    "East US",
			    "West US",
			    "West Europe",
			    "East Asia",
			    "South East Asia"
			    ]
		    }
	    },
	    "variables": {
	    },
	    "resources": [
		    {
			    "name": "[parameters('PrimaryStorageName')]",
			    "type": "Microsoft.ClassicStorage/StorageAccounts",
			    "location": "[parameters('PrimaryStorageLocation')]",
			    "apiVersion":  "2015-06-01",
			    "dependsOn": [ ],
			    "properties": {
			    	"accountType": "[parameters('PrimaryStorageType')]"
			    }
		    }
	    ],
	    "outputs": {
	    }
    }

<!--excerpt-->

## References

- [Microsoft Azure - Understand differences between Resource Manager and classic deployment models](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/)
- [Stack Overflow - Add storage to Azure resource manager](http://stackoverflow.com/questions/27347200/add-storage-to-azure-resource-manager)
- [Stack Overflow - powershell - How to force Azure Storage Account as classic](http://stackoverflow.com/questions/31886601/how-to-force-azure-storage-account-as-classic)
