---
layout: post
title: Publishing Assemblies into the GAC without GacUtil
category: Deployment
---

I constantly find myself googling this code snippet - its nice to keep handy: 

    [System.Reflection.Assembly]::Load("System.EnterpriseServices, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
    $publish = New-Object System.EnterpriseServices.Internal.Publish
    $publish.GacInstall("c:\temp\publish_dll.dll")

<!--excerpt-->

## References

- [MSDN - Publish.GacInstall Method (System.EnterpriseServices.Internal)](https://msdn.microsoft.com/en-us/library/system.enterpriseservices.internal.publish.gacinstall.aspx)
- [The Technical Adventures of Adam Weigert - PowerShell: Install-Gac (GACUTIL for PowerShell)](http://weblogs.asp.net/adweigert/powershell-install-gac-gacutil-for-powershell)
- [StackOverflow.com - Deploy multiple dll files into gac without gacutil](http://stackoverflow.com/questions/24950268/deploy-multiple-dll-files-into-gac-without-gacutil)
