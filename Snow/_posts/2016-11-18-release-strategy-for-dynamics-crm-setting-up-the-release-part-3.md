---
layout: post
title: Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release
category: Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services
---

This is the final installment of the a three part series. Use to following links to access part 1 and 2:

- [Release Strategy for Dynamics CRM - Part 1 - Preparation](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- [Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2)
- Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release


In the previous two posts I show you how to create the project structure and create a basic team build for your CRM solution artifacts. In this final installment I go about setting up a release definition for the build that we created.

1. Let's start off by creating a new release by navigating to the `Build and Release > Release` in your team project.
1. Select the `Empty` template option and click the next button.

	![Empty release template](/images/posts/CrmReleasePt3/10_ReleaseDefinition.png)

1. On the next step, choose the source for this release - we'll be using the [build definition that we created in part 2](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2). Once ready click the `Create` button.

	![Select build defintion](/images/posts/CrmReleasePt3/20_SelectBuildDefinition.png)

1. Now lets declare the variables that would be used later in the process. Navigate to the `Variables` tab and define the following variables:

	- `DevDeployLogin` - The username that used to deploy the solution.
	- `DevDeployPassword` - The password for the same account.

	Make sure to define the `DevDeployPassword` as a secret by clicking the padlock icon next to the field.

	![Release Variables](/images/posts/CrmReleasePt3/100_SetupVariables.png)

	Read more about [build and release variables here](https://www.visualstudio.com/en-us/docs/build/define/variables).
<!--excerpt--> 

1. Next, create a new agent step by clicking the `Add tasks` and selecting `Add an agent phase` in the context menu.

	![Add agent phase task](/images/posts/CrmReleasePt3/30_AddAgentPhase.png)

1. Select a `PowerShell` in the task catalog and click the `Add` button.

	![Add agent phase powershell](/images/posts/CrmReleasePt3/40_AgentPhasePowerShell.png)

1. Once the task is added, set the type to `Inline Script` and add the following script in the `Inline Script` field:

    `Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force -Scope CurrentUser`

	The purpose of this script is to install the NuGet provider on the agent. This is a prerequisite of the `Install-Module` command that will be used by the `Deploy-CrmSolution` script.

	And while we're here, let's rename the task to a more appropriate one. This would help us to distinguish it better.

	![Configure environment prerequisite powershell](/images/posts/CrmReleasePt3/50_SetupEnvironmentPreReqStep.png)

1. Now, add a second PoweShell script. This time set the type as `File Path` and rename the task to `Deploy CRM Solution`. This task would be performing the actual deployment to CRM.

	![Add CRM deploy step](/images/posts/CrmReleasePt3/60_CrmDeployStep.png)

1. Click the ellipsis on the `Script Path` field to browse and find the `Deploy-CrmSolution.ps1` script. 

	`$(System.DefaultWorkingDirectory)/DevBuild/drop/Nullfactory.Xrm.Tooling/Scripts/Deploy-CrmSolution.ps1`

	![Configure CRM deploy step script](/images/posts/CrmReleasePt3/70_CrmDeployStepScript.png)

1. Set the following arguments for the script - notice that the previously defined variables `DevDeployLogin` and `DevDeployPassword` being used as part of the script.
	`-serverUrl "https://servername.crm6.dynamics.com" -username "$(DevDeployLogin)" -password "$(DevDeployPassword)" -solutionName "Demo.AlphaSolution" -publishChanges -activatePlugins`

	![CRM deploy step final](/images/posts/CrmReleasePt3/90_FinalCrmDeployStep.png)

1. Expand the `Advanced` tab and click the ellipsis button on the `Working Folder` field. Set the working folder to the `Nullfactory.Xrm.Tooling\Scripts` folder on the resulting dialog. I do this in order to minimize the chance of encountering any long file path issues.

	![Configure CRM deploy step target](/images/posts/CrmReleasePt3/80_CrmDeployStepTarget.png)

1. Now that everything is setup, time to test out our new release definition. Let's queue a new release.

	![Create new release](/images/posts/CrmReleasePt3/110_CreateRelease.png)
	
1. Ensure that the release is being deployed and that it completed successfully.

	![Verify release status](/images/posts/CrmReleasePt3/120_ReleaseStatusInProgress.png)

	![Verify release status complete](/images/posts/CrmReleasePt3/130_ReleaseStatusComplete.png)


That's it folks. That's the end of the three part series showing how to setup a project structure, configure a build and finally release a source-controlled CRM solution.


## References

- [Create a release in Microsoft Release Management for Visual Studio Team Services and Team Foundation Server 2015](https://www.visualstudio.com/sl-si/docs/release/managing-releases/create-release)
- [Use build variables](https://www.visualstudio.com/en-us/docs/build/define/variables)
- [Use a PowerShell script to customize your build process](https://www.visualstudio.com/en-us/docs/build/scripts/)
- [How to deploy? Microsoft Release Management Tasks for Visual Studio Team Services and Team Foundation Server 2015](https://www.visualstudio.com/en-us/docs/release/author-release-definition/understanding-tasks)