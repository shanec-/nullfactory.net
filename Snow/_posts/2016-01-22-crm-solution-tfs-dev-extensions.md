---
layout: post
title: Strategy for Maintaining CRM Solutions in Team Foundation Server with Dynamics CRM Developer Extensions
category: Dynamics CRM, Team Foundation Server, Application Lifecycle Management
published: draft
---

This is a continuation of my previous post on building and maintaining a CRM solution in source control. In this version I replace the `Microsoft.Xrm.Data.Powershell` powershell module with the [Dynamics CRM Developer Extensions](https://visualstudiogallery.msdn.microsoft.com/0f9ab063-acec-4c55-bd6c-5eb7c6cffec4) extension for Visual Studio. 

This is a free alternative to the developer tools shipped with the CRM 2011 / 2013 SDKs and automates a lot of the manual steps during the development. The extension would be installed on each developer's Visual Studio instance. 

Under the hood, the extension makes use of the official CRM SDK to aid in its functionality. Specifically the plugin registration tool - which provides the  convenience of deploying plugins right from within the Visual Studio IDE and the solution packager which allows to pack and unpack CRM solutions. Therefore the CRM SDK is required to be present on each of the machines as well. 

Similar to my previous attempt, I will be creating a tooling project that hosts the tools. However this time it would only contain the `Microsoft.CrmSdk.CoreTools` the package. 

While I see that 

Exporting hte 

The only caveat


## Setting up the Dynamics CRM Developer Extensions

Perform the following steps on each of the developer machines:

1. [Download](https://www.microsoft.com/en-us/download/details.aspx?id=50032) and extract the CRM SDK into a folder that outside of the intended project solution.
2. [Download](https://visualstudiogallery.msdn.microsoft.com/0f9ab063-acec-4c55-bd6c-5eb7c6cffec4) and install the Dynamics CRM Developer Extensions.
2. Open Visual Studio and navigate to `Tools > Options`.
3. Navigate to the `CRM Developer Extensions` node.
4. Update the folders locations for the `Plug-in Deployer` and `Solution Packager` nodes to point to the `SDK\bin` and `SDK\Tools\PluginRegistration` folders respectively.

## Setup the Tooling Project

1. Create a new `Nullfactory.Crm.Tooling` class library.
2. Install the `Microsoft.CrmSdk.CoreTools` using the following command: 

	Install-Package Microsoft.CrmSdk.CoreTools -Project Nullfactory.Crm.Tooling

3. Remove the default `class1.cs` and `AssemblyInfo.cs` files. 
4. (Optional) Edit the `csproj` file and remove the properties folder.
5. Update the build configuration to prevent the project from being built. 

## Setup Solution Project and Unpacking Existing Solution

1. Create a new project for the CRM solution using a `CRM Solution Package` template.
2. Right click on the newly created project and navigate to the `CRM Developer Extensions > Solution Packager` context menu.
3. Connect to the CRM instance and select the solution.
4. Click on the `Unpackage Solution` button to automatically unpack and add the files into the project.
5. Next edit the csproj file and add the following msbuild task to automatically package the solution whenever the solution is built.

	<Target Name="Build">
		<Exec Command="$(SolutionDir)\Nullfactory.Crm.Tooling\bin\coretools\SolutionPackager.exe /action:pack /packagetype:both /folder:$(MSBuildProjectDirectory) /zipfile:$(OutDir)$(MSBuildProjectName).zip" />
	</Target>

## Final Thoughts

You might be wondering as to why we need the tooling project when the SDK exists. The reasoning for this was to reduce the footprint of the project for TFS team builds. The full fledged CRM SDK is around 200+ MB, while the core tools sit at just under 8. And since the builds only really requires the `SolutionPackager.exe` and dependent assemblies, the core tools are quite sufficient. The only drawback I can foresee is the possibility of the version of the Solution Packager contained in the coretools may go out of sync with the ones in the developer boxes.

## References

- [Dynamics CRM Developer Extensions](https://visualstudiogallery.msdn.microsoft.com/0f9ab063-acec-4c55-bd6c-5eb7c6cffec4)
- [Download Microsoft Dynamics CRM Software Development Kit (SDK) for CRM Online and on-premises CRM 2016 from Official Microsoft Download Center](https://www.microsoft.com/en-us/download/details.aspx?id=50032)