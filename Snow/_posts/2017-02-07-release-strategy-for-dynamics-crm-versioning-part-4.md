---
layout: post
title: Release Strategy for Dynamics CRM - Part 4 - Versioning
category: Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, Team Foundation Server, ALM, Git 
---
In this post I detail my approach to implement an automated versioning strategy in CRM solutions and its custom assemblies with the help of Visual Studio Team Services.

Although I originally only intended it to be a 3 part series I think I will continue to add to it as I keep making improvements. To recap, I previously prepared the project structure to host a CRM solution with the aid of the `nullfactory-xrm` [generator](https://www.npmjs.com/package/generator-nullfactory-xrm) and created a build and release definitions for it. You can find the previous posts here:

- [Release Strategy for Dynamics CRM - Part 1 - Preparation](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- [Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2)
- [Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/)
- Release Strategy for Dynamics CRM - Part 4 - Versioning

There were two aspects of versioning that I wanted to automatically increment with each release. Firstly the CRM Solution itself and then the custom assemblies included as part of the solution. I also wanted the flexibility to version the two independently of each other (I don't think its a very common scenario, but I would still like to have it as an available option).

As with my previous posts, the instructions in this are specific to Visual Studio Team Services (VSTS), but can be adapted to work with other build and release systems. 

<!--excerpt--> 

## Pre-requisites

- Ideally, a solution structure generated using version 1.3 (or better) of the `nullfactory-xrm` generator as the template includes the versioning script as part of it - for the sake of simplicity, the post assumes that this is true. [More Info.](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- All solution and custom code artifacts and checked-in into source control in VSTS. 
- Basic build definition that builds the artifacts and generates a CRM solution using the `SolutionPackager.exe`. [More info](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2)


## Integrating with the Team Build

1. Start off by creating a standard build definition for the solution as described in [Part 2](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2). 

	![Build Definition](/images/posts/CrmReleasePt4/10_BuildDefinition.png)

1. Next we need to define a pair of variables for each set of version numbers we want to maintain. In this scenario I only want one version number to be shared between the custom assemblies and CRM solution. Let's do this by navigating to the `Variables` tab and creating the `SuperSolutionMajorVersion` and `SuperSolutionMinorVersion` variables.

	![Define Build Variables](/images/posts/CrmReleasePt4/20_DefineBuildVariables.png)

1. I want to keep the artifact build number consistent with the name of the actual build, so navigate to the `General` tab and set the `Build number format` to `$(SuperSolutionMajorVersion).$(SuperSolutionMinorVersion).$(Build.BuildId).0`. 

	Notice that I intentionally hard coded the release version to `0`. Tweak this format to suit your own needs, but ensure that the number generated is compatible with the versioning scheme used by CRM - *major.minor.build.release* or *year.month.day.revision*

	![Set Build Number Format](/images/posts/CrmReleasePt4/30_SetBuildNumberFormat.png)

1. Create a PowerShell task which invokes the `ApplyVersionToArtifact.ps1` against the assemblies.

    .\Nullfactory.Xrm.Tooling\scripts\ApplyVersionToArtifact.ps1
    
    ApplyVersionToAssemblies -BuildSourcePath $(Build.SourcesDirectory) -BuildBuildNumber "$(SuperSolutionMajorVersion).$(SuperSolutionMinorVersion).$(Build.BuildId).0"

	![Apply Assembly Version PowerShell](/images/posts/CrmReleasePt4/40_AssemblyPowerShell.png)

1. Similarly create a second PowerShell task that applies the version number to the CRM Solution.

	. .\Nullfactory.Xrm.Tooling\Scripts\ApplyVersionToArtifact.ps1
	
	ApplyVersionToCrmSolution -BuildSourcePath $(Build.SourcesDirectory) -BuildBuildNumber "$(SuperSolutionMajorVersion).$(SuperSolutionMinorVersion).$(Build.BuildId).0"

	![Apply Crm Solution Version PowerShell](/images/posts/CrmReleasePt4/50_SolutionPowerShell.png)

1. Finally, lets queue a new build to make sure that everything works as expected. In the Queue build dialog, manually provide a Major and Minor version for the new release. The Major and Minor versions would act as the "public product" number, were as the build number would be the actual iteration.

	![Queue New Build](/images/posts/CrmReleasePt4/55_QueueBuild.png)

1. Verify that the build completed successfully - notice that it created using new build number formatting.

	![Build Results](/images/posts/CrmReleasePt4/60_BuildResult.png)

1. Open up the packaged solution and verify that the `Solution.xml` has the updated version number. Also ensure that the assembly within the solution has been updated as well.

	![Empty release template](/images/posts/CrmReleasePt4/70_SolutionXml.png)

## Final Thoughts

While there were many implementations on the web I based mine off this one [https://github.com/cilerler/rupen/blob/dea035c6e6158abc7a6e449b931ae316c0ef2eb8/build.vso/ApplyVersionToAssemblies.ps1](https://github.com/cilerler/rupen/blob/dea035c6e6158abc7a6e449b931ae316c0ef2eb8/build.vso/ApplyVersionToAssemblies.ps1). 

If the project structure was not generated using the `nullfactory-xrm` generator, then [download the latest version](https://github.com/shanec-/generator-nullfactory-xrm/blob/master/generators/app/templates/Nullfactory.Xrm.Tooling/Scripts/ApplyVersionToArtifact.ps1) of the `ApplyVersionToAssemblies.ps1 `script into your own project structure. 

The versioning script can be run manually on developers machine, but its most effective when used together with an automated build and release suite like VSTS. 

I had considered using a tokenization as a possible approach to replacing the version numbers, but the problem with that is that `solution packager` does not like tokens embedded in the `solution.xml` file. This means that a developer cannot build the solution in their own development environment.

## References

- [rupen/ApplyVersionToAssemblies.ps1 at dea035c6e6158abc7a6e449b931ae316c0ef2eb8 · cilerler/rupen](https://github.com/cilerler/rupen/blob/dea035c6e6158abc7a6e449b931ae316c0ef2eb8/build.vso/ApplyVersionToAssemblies.ps1)
- [Create patches to simplify solution updates](https://msdn.microsoft.com/en-us/library/mt593040.aspx)
- [Solution versioning best practice - Microsoft Dynamics CRM Community Forum](https://community.dynamics.com/crm/f/117/t/151258)
- [Hardcore CRM: Do a major upgrade of a plugin version! | Journey into CRM](http://journeyintocrm.com/archives/1092)
- [CRM 2013 – Understanding Solutions and how they work – Hosk's Dynamic CRM Blog](https://crmbusiness.wordpress.com/2014/03/31/crm-2013-overview-of-solutions/)
- [Chaminda's Blog: Apply Build Number to Assembly Version with New TFS Builds (Build vNext)](http://chamindac.blogspot.com.au/2015/12/apply-build-number-to-assembly-version.html)
- [TFS Build 2015 … and versioning! | Into ALM with TFS](https://intovsts.net/2015/08/24/tfs-build-2015-and-versioning/)
- [GitHub Gist - Update-AssemblyInfoVersionFiles.ps1](https://gist.github.com/pietergheysens/14d7d98547fe35470d0e)
- [CustomActivities/ApplyVersionToAssemblies.ps1 at master · tfsbuildextensions/CustomActivities](https://github.com/tfsbuildextensions/CustomActivities/blob/master/Source/Scripts/ApplyVersionToAssemblies.ps1)
- [Release Management Utility tasks - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.utilitytasks)
 

