---
layout: post
title: Release Strategy for Dynamics CRM - Part 5 - Deploy Third-Party Solutions
category: Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, Team Foundation Server, ALM, Git 
---
More often than not the custom Dynamics 365 solutions I develop depend on third party solutions to provide additional functionality. Therefore these solutions become a pre-requisite to the successful deployment of my own solution. 

In this post I go about updating the [previously setup project](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/), [build](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2) and [release](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/) definitions in Visual Studio Team Services (VSTS) in order to enable deployment of third party CRM solutions.
 
## Pre-requisites

- Project structure and deployment scripts based on [generator-nullfactory-xrm](https://www.npmjs.com/package/generator-nullfactory-xrm) (with a minimum version of at least 1.4.0).
- A working build and release definitions already setup.

Read more about setting up the project structure and build here:

- [Release Strategy for Dynamics CRM - Part 1 - Preparation](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- [Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2)
- [Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/)

## Project Structure

I start off by adding a solution level folder to hold third party solutions and  check-in the changes into source control.

![Project structure](/images/posts/CrmReleasePt5/10_structure.png)

<!--excerpt-->

## Build

Next, we update the build to ensure that the solutions are included as part of the build drop. I add a `Copy Files` task with the following parameters:
	
- `Source Folder`: `$(build.sourcesdirectory)`
- `Contents`: `**/_prereq-solutions/*.zip`
- `Target Folder`:	`$(build.artifactstagingdirectory)`

![Updated build definition](/images/posts/CrmReleasePt5/20_build.png)

I trigger a new build to make sure that the solutions are copied over as part of the build artifacts.

## Release

Now that our third-party solutions are included as part of the build drop, its time to deploy them. In this scenario I deploy all the solutions contained within the `_prereq-solutions` folder. 
 
I added a new PowerShell task as a precursor to the main solution deployment. I set it as an `inline script` with the following body:

	Get-ChildItem "..\..\_prereq-solutions\" -Filter *.zip | 
	Foreach-Object {
		.\Deploy-CrmSolution.ps1 -serverUrl "$(sndbxservername)" -username "$(sndbxusername)" -password "$(sndbxpassword)" -externalSolutionFileName $_.FullName -publishChanges -activatePlugins -importAsHoldingSolution:$false
	}

I also set the working folder to the `Nullfactory.Xrm.Tooling/Scripts` folder.

This script iterates through all the solutions in the folder and attempts to deploy them. 

![Updated release definition](/images/posts/CrmReleasePt5/30_release.png)

If you have multiple inter-dependent third-party solutions and require them to deployed in particular sequence you can either: 

- Add additional PowerShell tasks for each of the solutions.
- Rename the solutions with a ranking number so that the GetChild function orders and execute them implicitly. 