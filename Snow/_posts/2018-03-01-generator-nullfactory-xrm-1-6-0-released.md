---
layout: post
title: generator-nullfactory-xrm 1.6.0 Released!
category: Dynamics CRM, Dynamics CRM Online, Yeoman, npm, nodejs, generator-nullfactory-xrm
---
I just [released](https://www.npmjs.com/package/generator-nullfactory-xrm) version `1.6.0` of `generator-nullfactory-xrm`. This release includes the following changes:

![generator-nullfactory-xrm](/images/posts/generator160/hero.png)

- V9 Support!
- Simplified post installation steps with the introduction of `_RunFirst.ps1` script.
- Generate VSTS compatible YML build file:
   - Added `nullfactory-xrm:cibuild` sub-command.
- Generate individual CRM project file post initial scaffolding:
  - Added `nullfactory-xrm:solution` sub-command. 
- Default generator creates a tailored `README.md` file with a summary of changes.
- Fixed issue with generated workflow template code.
- CRM SDK assemblies updated:
  - Microsoft.CrmSdk.CoreTools `8.2.0.4` => `9.0.0.7`
  - Microsoft.CrmSdk.CoreAssemblies `8.1.0.2` => `9.0.0.7`
  - Microsoft.CrmSdk.Workflow `8.1.0.2` =>  `9.0.0.7`
- Added (Beta) version of the Online Management API scripts.

<!--excerpt-->

Please feel free to submit any feature requests or issues found to [https://github.com/shanec-/generator-nullfactory-xrm/issues](https://github.com/shanec-/generator-nullfactory-xrm/issues)

Read more about the Dynamics 365 release strategy and how everything fits together:

- [Release Strategy for Dynamics CRM - Part 1 - Preparation](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- [Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2)
- [Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/)
- [Release Strategy for Dynamics CRM - Part 4 - Versioning](/2017/02/release-strategy-for-dynamics-crm-versioning-part-4/)