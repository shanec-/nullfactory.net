---
layout: post
title: Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build
category: Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, Team Foundation Server, ALM
---

This is the second of a three part series in which I walk through setting up a release management strategy for Dynamics CRM.

- [Release Strategy for Dynamics CRM - Part 1 - Preparation](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build
- [Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/)
- [Release Strategy for Dynamics CRM - Part 4 - Versioning](/2017/02/release-strategy-for-dynamics-crm-versioning-part-4/)


In first part of the series accomplished the following:
	
1. Setup the project structure using the `nullfactory-xrm` yeoman generator.	
1. Download the CRM solution from the remote server and unpacked it using the solution packager tool.
1. Checked-in all the artifacts back into Visual Studio Team Services (VSTS) source control.

In this post I describe the steps required for setting up a team build. One of the goals with the generated project structure was to have it work with team builds with the least amount configuration. 

## Setting Up the Team Build

1. Navigate to the `Build and Release > Builds` from within the VSTS team project.

	![Build and Release Menue](/images/posts/CrmReleasePt2/10_NewDefintion.png)

1. Create a new build definition using a vanilla `Visual Studio` template.

	![Visual Studio Template](/images/posts/CrmReleasePt2/20_SelectBuildTemplate.png)

<!--excerpt-->  

1. If you want to use the artifact from this build as part a release deployment, then we need to explicitly copy the deploy scripts into the artifact staging folder. Let's do this by clicking on the `Add Task` and selecting `Copy Files` task and licking the `add` button. 

	![Copy Task](/images/posts/CrmReleasePt2/30_CopyTask.png)

	This explicit copy is required because the `Nullfactory.Xrm.Tooling` is not designated to build as part of the solution.

1. Next let's configure the task with the following parameters:
	
	- Source Folder -  `$(build.sourcesdirectory)`
	- Contents - `**/Deploy-*.ps1`
	- Target Folder - `$(build.artifactstagingdirectory)`
	- Change the name of the task into something more meaningful - `Copy Deploy Scripts to: $(build.artifactstagingdirectory)`

	![Copy Task](/images/posts/CrmReleasePt2/40_CopyTaskParameters.png)

1. Make sure that the newly added task is set to execute before the `Publish Artifact: drop` task.

1. Save and provide the new build a name.

	![Build Name](/images/posts/CrmReleasePt2/50_BuildName.png)

1. Now let's test it out by queuing a new build.

1. Once the build is complete, navigate to the artifacts and explore the drop folder to verify that all the required file have been copied .

	![Build Artifacts](/images/posts/CrmReleasePt2/60_BuildArtifacts.png)

Congratulations! Now you have team build that builds and packages the Dynamics CRM solution. Go forth and extend the definitions to suit your particular development workflow.

In the next post I will show you how to setup a release management definition and automate deployment to Dynamics CRM environments.