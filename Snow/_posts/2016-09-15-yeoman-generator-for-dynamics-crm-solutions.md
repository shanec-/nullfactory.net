---
layout: post
title: Yeoman Generator For Dynamics CRM Solutions
category: Dynamics CRM, Dynamics CRM Online, Yeoman, npm, nodejs, generator-nullfactory-xrm
---

I've previously experimented on creating a simplified way of [maintaining CRM solutions in source control](/2016/01/crm-solution-tfs-microsoft-xrm-data-powershell/). While I was quite happy with the end result, the process of setting up the projects, scripts and post-build steps was a little bit tedious. 

This is where Yeoman code generator comes in. Yeoman is a very popular javascript based scaffolding engine that enables setting up new projects a breeze. I created my own custom generator that would help me to kick start new CRM solutions quickly. You can find the end product here: [https://www.npmjs.com/package/generator-nullfactory-xrm](https://www.npmjs.com/package/generator-nullfactory-xrm)

## Installation

First, install [Yeoman](http://yeoman.io) and generator-nullfactory-xrm using [npm](https://www.npmjs.com/).

    npm install -g yo
    npm install -g generator-nullfactory-xrm

Then generate your new project:

	yo nullfactory-xrm

## Execution

Template questions and their purpose:

1. Visual Studio solution file name?  `The visual studio solution filename.`
2. Visual Studio project prefix?  `The prefix for the projects generated. This can be an organization name or preferred convention.`
3. Source CRM server url? `This is the source CRM server url. Example:[https://sndbx.crm6.dynamics.com](https://sndbx.crm6.dynamics.com)`
4. Source CRM Username? `The username used to connect to the source CRM server.`
5. Source CRM Password? `The password for the above account.`
6. Source CRM Solution Name? `The name of the CRM solution to be extracted.`
7. Add *.WebResource project? `Specifies if a new project should be created to manage the web resouces.`
8. Add *.Plugin project? `Specifies if a new plugin project should be created.`

<!--excerpt-->

## Post Installation Setup

Start off by updating the packages for the `Nullfactory.Xrm.Tooling` project. Open up the `Package Manager Console` in Visual Studio and execute the following command:

```
Update-Package -reinstall -Project Nullfactory.Xrm.Tooling
```

Next, install the [`Microsoft.Xrm.Data.PowerShell`](https://github.com/seanmcne/Microsoft.Xrm.Data.PowerShell) powershell module. On a Windows 10 or later, do this by executing the included powershell script `Nullfactory.Xrm.Tooling\_Install\Install-Microsoft.Xrm.Data.PowerShell.ps1` or manually running the following command:

```
Install-Module -Name Microsoft.Xrm.Data.PowerShell -Scope CurrentUser
```

## Syncing a Solution to the Project

Anytime the CRM solution needs to be synchronized back to the project, execute the script located at `Nullfactory.Xrm.Tooling\Scripts\Sync-CrmSolution.Param.ps1`.

### Resource Mapping

Edit the mapping file to map to the appropriate resource project. They are located in the `Nullfactory.Xrm.Tooling\Mapping` folder. 
More information on the structure of the mapping file can be found [here](https://msdn.microsoft.com/en-us/library/jj602987.aspx#use_command)

## Building the CRM Solution

The repackaging the extracted solution is integrated as a post-build step of the solution class library. Simply build it to output both a managed as well as unmanaged  CRM solution package. 

## Final Thoughts 

I think I did pretty good for my first generator and hope to continue to evolve it as I learn more about the templating engine.

Here are some ideas that I have floating around:
* The ability to add multiple CRM solutions in one go.
* The ability to rename the Plugin and WebResources Project
* Add new projects after the initial project structure has been generated.
* Refactor the script to better re-use components.

The generator is [hosted in GitHub](https://github.com/shanec-/generator-nullfactory-xrm). Please feel free to fork and hack away at it. I would also appreciate any feedback you might have. 

## References

- [Strategy for Maintaining CRM Solutions in Team Foundation Server using Microsoft.Xrm.Data.PowerShell](http://www.nullfactory.net/2016/01/crm-solution-tfs-microsoft-xrm-data-powershell/)
- [Writing Your Own Yeoman Generator | Yeoman](http://yeoman.io/authoring/)
- [Create A Custom Yeoman Generator in 4 Easy Steps | Scotch](https://scotch.io/tutorials/create-a-custom-yeoman-generator-in-4-easy-steps)