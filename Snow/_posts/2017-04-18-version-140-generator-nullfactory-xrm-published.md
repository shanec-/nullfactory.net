---
layout: post
title: Version 1.4.0 of generator-nullfactory-xrm Published
category:  Dynamics CRM, Dynamics CRM Online, Yeoman, npm, nodejs, generator-nullfactory-xrm
---
Version `1.4.0` of the generator released. It includes the following changes:

- Added support to deploy third-party CRM solutions.
- CRM SDK assemblies updated:
	- Microsoft.CrmSdk.CoreTools `8.1.0.2` => `8.2.0.4`
	- Microsoft.CrmSdk.CoreAssemblies `8.1.0.2` => `8.2.0.2`
	- Microsoft.CrmSdk.Workflow `8.1.0.2` => `8.2.0.2`
- If opted-in to generate plugins and workflows projects, the generator would automatically create mapping entries in the `mapping-xml` file. 
- PowerShell script updates:
	- Better handling of exception and failure flows.
	- `*.param.ps1` scripts to have high verbosity by default.
	- Fixed deploy script to have the new solution name. (i.e. `ProjectPrefix+SolutionName`) 
	- Improved help documentation.

<!--excerpt-->

Please feel free to submit any feature requests or issues found to https://github.com/shanec-/generator-nullfactory-xrm/issues