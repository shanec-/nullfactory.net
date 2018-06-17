---
layout: post
title: generator-nullfactory-xrm 1.7.0 Released!
category: Dynamics CRM, Dynamics CRM Online, Yeoman, npm, nodejs, generator-nullfactory-xrm
---
I just [released](https://www.npmjs.com/package/generator-nullfactory-xrm) version `1.7.0` of `generator-nullfactory-xrm`. This release includes a lot of defect fixes, dependency version upgrades and refactored code.

- Feature #37: Added ability to upgrade tooling PowerShell scripts via the generator.
    - Run `yo nullfactory-xrml:tooling`
- Feature #12: The generated *.csproj files now have unique identifiers. This should avoid any potential conflict between projects.
- Feature #35: .gitignore file updated to be compatible with nested "standard project structure". (see https://gist.github.com/davidfowl/ed7564297c61fe9ab814).
- Fixed #34: Pull-CrmSolution.ps1 now downloads the zip file to a temporary file location before extraction.
- Fixed #36: `solution` sub-generator now creates mapping file as expected.
- Updated plugin and workflow project dependencies:
    - Microsoft.CrmSdk.CoreAssemblies - `9.0.0.7` ==> `9.0.2.3`
    - Microsoft.CrmSdk.Worklow - `9.0.0.7` ==> `9.0.2.3`
- Refactored the internal structure of the generator.
- Refreshed package dependencies.
- Fixed broken unit tests.

<!--excerpt-->

Please feel free to submit any feature requests or issues found to [https://github.com/shanec-/generator-nullfactory-xrm/issues](https://github.com/shanec-/generator-nullfactory-xrm/issues)

Read more about the Dynamics 365 release strategy and how everything fits together:

- [Release Strategy for Dynamics CRM - Part 1 - Preparation](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- [Release Strategy for Dynamics CRM - Part 2 - Setting Up the Build](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2)
- [Release Strategy for Dynamics CRM - Part 3 - Setting Up the Release](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/)
- [Release Strategy for Dynamics CRM - Part 4 - Versioning](/2017/02/release-strategy-for-dynamics-crm-versioning-part-4/)