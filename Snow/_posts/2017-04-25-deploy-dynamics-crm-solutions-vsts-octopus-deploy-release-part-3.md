---
layout: post
title: Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 3 - Release and Deployment
category: Octopus Deploy, Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, ALM, Git 
---
In this third and final installment I cover the steps involved in setting up the minimum release and deployment steps required for a successful Dynamics CRM/365 deployment. The steps and actions performed in the previous posts acts as prerequisites for this one.

Related posts from the series:

- [Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 1 - Setup](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-setup-part-1/)
- [Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 2 - Build](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-build-part-2/)
- Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 3 - Release and Deployment

## The Packages

In the [previous post](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-build-part-2/), I setup the team build to automatically publish packages into the Octopus Deploy package repository. 

We can verify that they have indeed been published by navigating to the `Library > Packages` tab for a list of available packages.

![Available Packages](/images/posts/OctoDeployPt3/05_published_package.png) 

## Defining the Environment

Environments are a logical grouping of targets / machines used by deployments. For the purpose of this post, I will create one environment - `Testing` and it would contain a single CRM deployment target - the CRM Server). 

1. Let's start off by clicking on the `Environments` from the main menu to navigate to the environment configuration page.
1. Click on `Add environment` in the resulting page.
1. Set `Testing` as the name of the environment .

	![Existing Build Definition](/images/posts/OctoDeployPt3/10_create_environment.png)
1. Click on `Save` to confirm selection.

<!--excerpt-->

Next, define the deployment target for this environment:

1. Navigate to the Environments summary page and click the `Add deployment target` button.
1. Select `Cloud Region` as the deployment target.

	![Existing Build Definition](/images/posts/OctoDeployPt3/20_deployment_target.png)

1. Set `Test CRM Server` as the `Display name`.
2. Select the previously created `Testing` as one of the `Environments`.
2. Create a new role called `crm instance`.

	![Existing Build Definition](/images/posts/OctoDeployPt3/30_deployment_target2.png)

1. Click on `Save` button to confirm.

	![Existing Build Definition](/images/posts/OctoDeployPt3/40_deployment_target3.png)

### Variable Sets

I will be creating a variable set to host all settings required to connect to the CRM Server. It makes configuration easier and also provides us with the ability to reuse the same server in multiple projects.

1. From the main menu navigate to `Library > Variable Sets`
1. Click on `Add variable set`

	![Existing Build Definition](/images/posts/OctoDeployPt3/50_variables.png)

1. Define three new variables:
	
	- `#{TestCRM-servername}`: The CRM Server Url.  
 	- `#{TestCRM-username}`: The deployment username.
	- `#{TestCRM-password}`: The deployment password. On this one, set the `variable type` as `Sensitive`.

	![Existing Build Definition](/images/posts/OctoDeployPt3/60_variables_created.png)

1. Click the `Save` button.

### Define the Project

Projects allow to define a proper release lifecyle by bringing together the different environments and deployment steps. 

Let's define a new project:

1. Navigate to `Projects` menu.
2. Click on the `Add project`.
3. Provide a name and description. Leave the other options to default.
4. Click on `Save` to confirm selection.

	![Existing Build Definition](/images/posts/OctoDeployPt3/70_create_project.png)

Now associate our previously created variable set as part of our project:

1. Navigate to the `Variable > Library Variable Sets` tab within the project.
1. Click the `Include variable sets from the Library` button.
1. Choose the previously created variable set.

	![Existing Build Definition](/images/posts/OctoDeployPt3/80_include_variable_project.png)

1. Click the `Apply` and then `Save` buttons to confirm selection.

	![Existing Build Definition](/images/posts/OctoDeployPt3/90_include_variable_project2.png)

## Setup the Deployment Process 

The deployment steps are pretty simple and consists of two PowerShell scripts; the first one is an inline script that sets up the prerequisites while the second one does the actual deployment. 

### Prerequisite script 

1. Navigate to the `Process` tab within the project.
1. Click on `Add your first step`
1. Select a `Run a Script` template.

	![Existing Build Definition](/images/posts/OctoDeployPt3/100_step_template.png)

1. Update the following parameters while accepting the default selections for the others:

	- `Step name`: `Setup Prerequisites`
	- `Run on` : `Deployment targets`
	- `Runs on targets in roles` : `crm instance`
	- `Script source` : `Source code`
	- `Script`: `Powershell`
	
1. Provide the following for the body of the script:

		Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force -Scope CurrentUser

	![Existing Build Definition](/images/posts/OctoDeployPt3/110_step_1.png)

1. Click `Save` to apply changes.

### Deployment Script

The steps are very similar to the previous one, but this time we want to execute a script that is already part of the package. 

1. Once again,click on `Add step` and select a `Run a Script` template:
1. Update the following parameters while accepting the default selections for the others:

	- `Step name`: `Deploy CRM Solution`
	- `Run on` : `Deployment targets`
	- `Runs on targets in roles` : `crm instance`
	- `Script source` : `Script file inside a package`

1. Keep the `Package Feed` on `Octopus Server (built-in)` as we've already published our package into OD.
1. Set `Package ID` to `gnxdemo.crm`. This is the name that the build published.
1. Set the `Script file name` value to:

		Nullfactory.Xrm.Tooling\Scripts\Deploy-CrmSolution.ps1

1. Set the `Script parameters` value to by integrating the variables that we created earlier:
	
		-serverUrl "#{TestCRM-servername}" -username "#{TestCRM-username}" -password "#{TestCRM-password}" -solutionName "gnxdemo.crm" -publishChanges -activatePlugins

	![Existing Build Definition](/images/posts/OctoDeployPt3/120_step_2.png)

1. Click the `Save` button.

	![Existing Build Definition](/images/posts/OctoDeployPt3/130_step_summary.png)

## Release and Deployment

Now that we have everything setup let's create a new release and then deploy the same to the `Testing environment`.

1. Navigate to the Project page.
1. Click on the `Create Release`.
1. Provide a version number for the new release. I gave it `0.0.1`.
1. Ensure that the `Deploy CRM Solution` step is referencing the latest version of the solution.
1. Click on the `Save` button the confirm the release. The release page shows an overview of the release including the its lifecyle path and as well as the packages being deployed.
1. Click on the `Deploy to Testing` in order to deploy the solution to the testing environment.
2. Click the `Deploy now` button in the resulting page to kick off the deployment.

	![Existing Build Definition](/images/posts/OctoDeployPt3/140_deploy_release.png)

1.  The page would redirect to a summary page showing the progress of the deployment. 

	![Existing Build Definition](/images/posts/OctoDeployPt3/150_deploy_task_progress.png)

1. Click on the `Task Log` to view a more verbose view of the solution being deployed.	

	![Existing Build Definition](/images/posts/OctoDeployPt3/160_deploy_task_log.png)

## Final Thoughts

While I think I have barely scratched the surface of the features provided by Octopus Deploy, the process of writing this series of posts made me appreciate the intuitive structure of key concepts as well as the myriad of options available to support even the most complex of release life cycles. This series also gave me the opportunity to validate the flexibility of the project structure generated using the [generator-nullfactory-xrm](https://www.npmjs.com/package/generator-nullfactory-xrm) - with its ability to work with different DevOps tools.

## References

- [Environments - Octopus Deploy](https://octopus.com/docs/key-concepts/environments)
- [Projects - Octopus Deploy](https://octopus.com/docs/key-concepts/projects)
- [Machine Roles - Octopus Deploy](https://octopus.com/docs/key-concepts/machine-roles)
- [Deployment targets - Octopus Deploy](https://octopus.com/docs/deployment-targets)
- [Azure Cloud Service Deployment Targets - Octopus Deploy](https://octopus.com/docs/deployment-targets/azure-cloud-service-target)
