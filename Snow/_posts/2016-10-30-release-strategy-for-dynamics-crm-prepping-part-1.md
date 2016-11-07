---
layout: post
title: Release Strategy for Dynamics CRM - Part 1 - Preparation
category: Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, Git
---

Earlier this year I demonstrated the strategy I use for maintaining CRM solutions in source control. I favor the approach due to its ability to be used in team builds and automated releases. Since my original post, I have created a yeoman generator that allows to quickly scaffold the project structure for new projects. And now, in this series of posts, I will walk you through the steps in setting up a new project, creating a team build finally implementing a release strategy using Visual Studio Team Services. 

In order to automate the process we first need to make sure that the solution is maintained in source control. 

## Prerequistes

- Visual Studio Team System (VSTS) source control.
- `Yeoman` code generator with the `generator-nullfactory-xrm` installed globally. Installation instructions can be found [here](https://www.npmjs.com/package/generator-nullfactory-xrm).

While in this particular demo I use a VSTS team project running on top of Git source control along with VSTS Release Management for the deployment, the same steps can be adapted to be used with any other build and release system.


## Prepping the Solution

1. Create a Team Project and select `Git` as the version control.
1. Clone a working copy onto a local machine. 
1. Create a .gitignore file that excludes build output - let's use [GitHub's definitions for Visual Studio](https://github.com/github/gitignore/blob/master/VisualStudio.gitignore).
1. Run the `nullfactory-xrm` yeoman generator in order to scaffold the project structure. More detailed instructions on running the generator can be found [here](https://www.npmjs.com/package/generator-nullfactory-xrm).

	![nullfactory-xrm yo generator](/images/posts/CrmReleasePt1/10_YoGenerator.png)

<!--excerpt-->  

1. Open up the generated solution in Visual Studio.
1. Force re-install the `Microsoft.CrmSdk.CoreTools` package into the `Nullfactory.Xrm.Tooling` project. Do this by running the following command in the Package Manager Console.
	
	`Update-Package -reinstall -project "Nullfactory.Xrm.Tooling"`

	![Update-Package -reinstall](/images/posts/CrmReleasePt1/20_UpdatePackage.png)

1. Next, sign the the plugins and workflows projects with a new strong key. Use the following command to generate a new key pair.

	`sn -k key.snk`

	![Sign Assembly](/images/posts/CrmReleasePt1/30_SignAssembly.png)

1. Next, let's sync up the CRM solution by running the synchronize script located in the `Scripts` folder in the `Nullfactory.Xrm.Tooling` project. This script downloads the CRM solution from the remote server and then expand it using the `SolutionPackager.exe` tool.

	`.\Sync-CrmSolution.Param.ps1`

	![Sync-CrmSolution](/images/posts/CrmReleasePt1/40_SyncScript.png)

	If you encounter the following error when executing the script - it means that you have both PsGet and OneGet installed and its conflicting with each other. [Simply uninstalling PsGet in order to resolve the conflict](https://til.secretgeek.net/powershell/psget_conflicts_with_PowerShellGet.html).

	![Install-Module Error](/images/posts/CrmReleasePt1/50_InstallModuleError.png)

1. Before proceeding any further, let's verify that the crm solution project can be built and solution zip file is the output.

1. Stage all the generated files into source control.

	`git add .`

1. Next, let's force stage the `coretools` folder. Due to the way the CoreTools package installs itself it is necessary to explicitly include these files into source control for the team build to work.

	`git add --force Nullfactory.Xrm.Tooling/bin/coretools/*.*`

1. Finally commit the changes.

	`git commit -m "initial commit"`

	![Install-Module Error](/images/posts/CrmReleasePt1/50_InstallModuleError.png)

In the next post I will go into setting up a team build for this project.

## References

- [psget conflicts with PowerShellGet · Today I Learned (secretGeek)](https://til.secretgeek.net/powershell/psget_conflicts_with_PowerShellGet.html)
- [StackOverflow - What is Install-Module command in Powershell? - Super User](http://superuser.com/questions/996417/what-is-install-module-command-in-powershell)
- [Installing a PowerShell Module](https://msdn.microsoft.com/en-us/library/dd878350(v=vs.85).aspx)
- [generator-nullfactory-xrm](https://www.npmjs.com/package/generator-nullfactory-xrm)