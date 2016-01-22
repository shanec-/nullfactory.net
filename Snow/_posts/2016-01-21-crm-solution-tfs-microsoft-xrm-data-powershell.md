---
layout: post
title: Strategy for Maintaining CRM Solutions in Team Foundation Server using Microsoft.Xrm.Data.PowerShell
category: Dynamics CRM, Team Foundation Server, ALM
---

I've come across a few strategies on the internet on achieving this goal, with each one having its pros and cons. This post describes my own attempt at setting up a development workflow that can be integrated with the team builds.

This implementation at its core revolves around the [`Microsoft.Xrm.Data.PowerShell`](https://github.com/seanmcne/Microsoft.Xrm.Data.PowerShell) module and the `Solution Packager` tool provided with the official SDK. I have minimized the use of 3rd party libraries and have intentionally excluded the use of any visual studio extensions or helper tools.

## Setting up the Tools

### Installing Microsoft.CrmSdk.CoreTools

Let's start off by creating a "tooling" project that would act as a hosts for the tools and scripts involved in the process.

1. Create a class library project to host the tools using for building the packages. I named mine `Nullfactory.Crm.Tooling`.
2. Install the `Microsoft.CrmSdk.CoreTools` nuget package by running the following command in the package manager console: 
	
	`Install-Package Microsoft.CrmSdk.CoreTools -Project Nullfactory.Crm.Tooling`

	This nuget package includes the `SolutionPackager.exe` tool.

	![Install Crm Core Tools](/images/posts/CrmDataPowershell/10_InstallCoreTools.png)

3. Next, clean up the project by removing the default `Class1.cs` file as well as the `Debug` and `Release` within the `bin` folder.
3. Update the solution configurations to not build this project. Do this by navigating to the solution property pages > `Configuration Properties` and un-ticking the check box against the build column.

	![Do not Build Project](/images/posts/CrmDataPowershell/20_DontBuild.png)

<!--excerpt-->

### Install Microsoft.Xrm.Data.PowerShell

The `Microsoft.Xrm.Data.Powershell` module makes interacting with CRM so much easier - it is used to connect to CRM and export the solutions. 

Follow these steps to install it:

1. Download the latest release from [here](https://github.com/seanmcne/Microsoft.Xrm.Data.PowerShell/releases/). Detailed installation instructions can be found on their github page. 
2. Ensure that the zip file is unblocked *before extracting the contents*. Once extracted, add them into the bin folder as part of the tooling project.

	![Unblock Zip Before Extraction](/images/posts/CrmDataPowershell/30_UnblockZip.png)
 
3. Since this module would have to be installed on each of the developer machines, I created a helper script that automates it - `Install-Microsoft.Xrm.Data.Powershell.ps1`. Add this as part of the project as well. 

	![Install Microsoft.Xrm.Data.Powershell](/images/posts/CrmDataPowershell/40_InstallPowershell.png)

	[Download the install script here](https://github.com/shanec-/Crm-PowershellBuildDemo/blob/master/src/Nullfactory.Crm.Tooling/bin/Install-Microsoft.Xrm.Data.Powershell.ps1).

## Setting up the Projects

Next, lets create class library projects for each of the CRM Solutions. This makes visualizing and managing the solution from within Visual Studio easier. And more importantly, give us the ability to add a msbuild tasks.

Edit the `csproj` file of the newly created project and add the following ms build task. 

	<Target Name="Build">
		<Exec Command="$(SolutionDir)\Nullfactory.Crm.Tooling\bin\coretools\SolutionPackager.exe /action:pack /packagetype:both /folder:$(MSBuildProjectDirectory) /zipfile:$(OutDir)$(MSBuildProjectName).zip" />
	</Target>

This ensures that both unmanaged and managed versions of the CRM solution is packaged anytime the project is built. 

Optionally, remove the `properties` folder and `AssemblyInfo.cs` file as they will not be required for these projects.

### Add Solution Export and Synchronization Script

Add the `Sync-CrmSolution.ps1` and `Sync-CrmSolution.Param.ps1` files into the tooling project. These scripts can be downloaded [here](https://github.com/shanec-/Crm-PowershellBuildDemo/blob/master/src/Nullfactory.Crm.Tooling/bin/Sync-CrmSolution.ps1) and [here](https://github.com/shanec-/Crm-PowershellBuildDemo/blob/master/src/Nullfactory.Crm.Tooling/bin/Sync-CrmSolution.Param.ps1).

![Installed Synchronization Scripts](/images/posts/CrmDataPowershell/60_SyncScriptsInstalled.png)

The `Sync-CrmSolution.ps1` script handles the exporting of the solution and performs the following actions:
 
 1. Deletes all the artifacts from the CRM solution project folder.
 2. Connects to the organization and exports both the managed and un-managed versions of the solution.
 3. Finally, unpacks them into the previously emptied folder.

The `Sync-CrmSolution.Param.ps1` script acts as a controller script with the actual parameters. Each developer would update this script to point to their own development CRM organization.

## Unpacking and Synchronizing 

Next, we unpack the initial version of the solution into the project. This is done using the following steps: 

1. Ensure that the `Sync-CrmSolution.Param.ps1` script is pointing to a valid CRM organization and solution and execute the script.

	![Extracting Solution](/images/posts/CrmDataPowershell/50_ExtractingSolution.png)
 
2. Once the solution has been unpacked, add the new artifacts into the project.
	
	![Crm Solution Extracted](/images/posts/CrmDataPowershell/70_SolutionExtracted.png)

3. Check-in all the changes done so far.

Now that the initial version is in source control, it raises the problem of figuring out the files that have changes in subsequent extractions. How do you figure out which files have changed so that only those files are check-in?

One of my colleague introduced me to an interesting technique that hes been using for a while. I like it a lot as its simple, effective and avoids having to do any explicit TFS integration.   

This method leverages Microsoft Visual Studio Team Foundation Server 2015 Power Tools in order to identify the files changed in the working folder. It requires each developer [install](https://visualstudiogallery.msdn.microsoft.com/898a828a-af00-42c6-bbb2-530dc7b8f2e1) it on their development box.

Whenever a developer synchronizes their version of the solution using the `Sync-CrmSolution.Param.ps1` script, the power tools would automatically detect and check out the edited files. One would still have to manually include new and deleted files into the project via the detected changes dialog, but that's a minor inconvenience I can live with. 

A positive side effect of this method is that we no longer have to be concerned about the `allowDelete` and `allowWrite` parameters in the Solution Packager tool.

## Final Thoughts

Now any time the class libraries hosting the solutions are built, the output would be packaged zip files for both managed and un-managed CRM solutions. 

Although I did not setup up separate projects for plugins and web resources in this example, it is certainly possible.  

I might also explore converting the entire `Nullfactory.Crm.Tooling` project into a template in a future post. It should make it a lot more easier integrate it into new projects.

Finally, I feel that the day-to-day operation part of this process is a little bit tedious and does not offer any significant advantage over the convenience of using an Visual Studio extension. I knew this going in, but I wanted to have an understanding of the work involved in setting this up. 

## References
- [Releases · seanmcne/Microsoft.Xrm.Data.PowerShell](https://github.com/seanmcne/Microsoft.Xrm.Data.PowerShell/releases/)
- [Use the SolutionPackager tool to compress and extract a solution file](https://msdn.microsoft.com/en-us/library/jj602987.aspx)
- [Installing Modules](https://technet.microsoft.com/en-us/library/dd878350(v=vs.85).aspx)
- [PowerShell - How to create a PSCredential object - Kotesh Bandhamravuri - Site Home - MSDN Blogs](http://blogs.msdn.com/b/koteshb/archive/2010/02/13/powershell-creating-a-pscredential-object.aspx)
- [Integrating SolutionPackager into Visual Studio - YouTube](https://www.youtube.com/watch?v=65MVXzMAWyg)
- [Dynamics CRM Parallel Development with Solution Packager | Wael Hamze](http://waelhamze.com/2014/01/12/dynamics-crm-parallel-development-with-solution-packager/)
- [Microsoft Visual Studio Team Foundation Server 2015 Power Tools extension](https://visualstudiogallery.msdn.microsoft.com/898a828a-af00-42c6-bbb2-530dc7b8f2e1)