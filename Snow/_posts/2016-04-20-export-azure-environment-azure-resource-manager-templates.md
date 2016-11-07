---
layout: post
title: Clone an Existing Azure Environment using Azure Resource Manager Templates
category: Azure, Azure Resource Manager
published: draft
---

I've been a big fan of the Azure Resource Manager framework for a while now and use it [religiously to source control and deploy](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/) my Azure environments. That being said, one of the features that I missed sorely was the the ability to export an existing environment back into a template - and I was not the only one who [wanted this feature](https://azure.microsoft.com/en-us/blog/export-template/). Microsoft has now included this in the recent release.

### Pre-requisites

Ensure that the latest version of the Azure SDK - Version 2.9 at the time of writing this post. 
Although it is not required to exporting a template using Azure web portal, it is useful when:

* Exporting or re-importing a template back into Azure using the `Azure PowerShell` module.
* Integrating with Visual Studio and maintaining the environment as a `Azure Resource Group` project.

### Export Existing Environment

While there are [three new ways to export](https://azure.microsoft.com/en-us/blog/export-template/) an environment, I would think that web based template export or the command line export would be the most common.

1. Log into the Azure Portal

or use the following command in an `Azure PowerShell` command prompt.

### Integrate with Azure Resource Manager Project



### Deploy

While I have experienced some of its [teething](/2015/10/deploy-classic-storage-azure-resource-manager/) [issues](http://stackoverflow.com/questions/32569564/azure-resource-manager-deployment-vs-classic-deployment-of-storage-accounts) before, the whole ecosystem and tools have been maturing nicely with each milestone. And I love the fact that Microsoft is continuing to listen to its customers' feedback and making improvements its ecosystem.

## References
 - [Resource Manager and classic deployment | Microsoft Azure](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/)
 - [Announcing template export feature in Azure Resource Manager | Blog | Microsoft Azure](https://azure.microsoft.com/en-us/blog/export-template/)
 - [Ability to export a Resource Group as a template – Customer Feedback for Microsoft Azure](https://feedback.azure.com/forums/223579-azure-portal/suggestions/7163577-ability-to-export-a-resource-group-as-a-template?page=1&per_page=20)
 - [Stack Overflow - Azure Resource Manager Deployment vs Classic Deployment of Storage Accounts](http://stackoverflow.com/questions/32569564/azure-resource-manager-deployment-vs-classic-deployment-of-storage-accounts)
 - https://azure.microsoft.com/en-us/documentation/articles/resource-group-template-deploy/