---
layout: post
title: Obfuscating a SharePoint-Integrated SSRS DataSource's Connection String  
category: SharePoint, SQL Server Reporting Services, Code Access Security
published: draft
---

When SQL Server Reporting Services (SSRS) is deployed as an SharePoint integrated solution, it enables a lot of its functionality to be managed right from within SharePoint. Starting from the 2013 version, the integration between SharePoint and SQL Server Reporting Services 2012 is more tightly coupled than its previous iterations. 

One of features in integrated mode is the ability to have the data sources (.rsds) and report files (.rdl) within a document library itself. This means that reports can reference a DataSource within any document library in the SharePoint site. 

In order for the report to work the user should have read permission on both the data source as well as the report file. The problem with this is that the same user can now potentially view the settings within the data source file, including the connection string.

## The Solution

The solution I came up with is to obscure the connection string by encrypting it.  I achieved this using the following steps:

1. Force the reports to get the connection string by evaluating an expression embedded within itself. 
2. Within this expression, call some custom code which manages the retrieval and decryption of the connection string. In my example, below I will be retrieving it from a configuration list stored in the same SharePoint server.

One of the limitations with this method is that you can no longer use a shared data source and each report has to have its credentials embedded.

### Create the report extensions assembly

Here's a summary of creating the report extension, link to the full source at the bottom of the post.

1. Create new a class library which contains your custom code.
1. Sign the assembly with a key as assembly would be deployed into the Global Assembly Cache. 
1. In order for the report server to call the custom code, the strong-named assembly must be marked with the `[assembly: AllowPartiallyTrustedCallers]` attribute. 

	![Build Process Parameter](/images/posts/ObfuscateSSRS/10_AllowPartiallyTrustedCallers.png)

1. Create a new public class and method that would retrieve the connection string. Add a `[SecuritySafeCritical]` attribute to the method. This method would take in one string parameter; the report server url. The report server url would be parsed and later used when generating the REST call to SharePoint.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/20_CodeGetConnectionString.png)

1. Create a method that retrieves the Default Credentials. Decorate it with the `[SecuritySafeCritical]` attribute as well. Explicitly assert the `EnvironmentPermission(EnvironmentPermissionAccess.Read, "USERNAME")` permissions before accessing the `CredentialStore`.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/50_CodeGetSecurityCredentials.png)

1. Create the method that makes the REST call to retrieve the obfuscated connection string from a SharePoint configuration list.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/40_CodeGetWebPermission.png)

1. The `EnvironmentPermission` assertion operation has to be in its own method with its own `[SecuritySafeCritical]` attribute in order for it to work together with `WebPermission` assertion, otherwise the following exception would be thrown:

		Exception: System.Security.SecurityException Exception Message: Stack walk modifier must be reverted before another modification of the same type can be performed. Stacktrace:    at System.Security.CodeAccessSecurityEngine.Assert(CodeAccessPermission cap, StackCrawlMark& stackMark)
		   at System.Security.CodeAccessPermission.Assert()

	After numerous failed attempts at accessing the SPList via the SharePoint object model (I kept running into exceptions stating that I required full trust), I find that partial trust mode is [no longer supported and has been deprecated](https://msdn.microsoft.com/en-us/library/office/dn268593.aspx). As I was unable to fnd anyone who had successfully got it to work, I opted to access the list via the REST service with its lesser dependencies and security permission requirements. 

1. Compile and add the new assembly into the Global Assembly Cache.

### Setting up the Report

1. Open up a copy of your report using the [latest version of Report Builder](http://www.microsoft.com/en-us/download/details.aspx?id=29072). 
1. Right click on the work space area outside of the report to open up the report context menu. Click on the `Report Properties...`. This opens up the `Report Properties` dialog.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/60_ReportPropertiesContextMenu.png)

	![Build Process Parameter](/images/posts/ObfuscateSSRS/70_ReoprtPropertiesDialog.png)

1. Navigate to the Code tab and add an entry for our custom assembly.
1. Next, create a new entry for the class that would handle the logic. Provide an instance name that would be used to access our custom code.  

	![Build Process Parameter](/images/posts/ObfuscateSSRS/80_ReportPropertiesDialog.png)

1. Now that our declarations have been done, edit the report data source.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/90_EditDataSource.png)

1. Click on the `fx` button to open up the expressions dialog.
	
	![Build Process Parameter](/images/posts/ObfuscateSSRS/100_ConnStringSetExpression0.png)

1. Edit the expression and replace it to use our custom method. Pass in `Globals!ReportServerUrl` as the parameter.
	
	![Build Process Parameter](/images/posts/ObfuscateSSRS/110_ConnStringSetExpression1.png)

1. Upload the updated report into the SharePoint report list. If you already have a report with the same name and with a shared data source linked to it, your new connection details will be replaced. In this case, delete your existing report and re-upload the new one. 

1. Next, edit the Report Data Source by selecting the `Manage Data Sources` option in the report file context menu. And ensure that connection type is set to `Custom data source` and connection string to `Use connection string expression defined in the report`.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/130_ManageDataSource.png)

	![Build Process Parameter](/images/posts/ObfuscateSSRS/120_ConnStringNotInitialized.png)

1. For the credentials, I will be using stored credentials for my scenario. I noticed that the `Test Connection` button does not work when if the connection string needs to be evaluated.

## Final Thoughts

* The [source](https://github.com/shanec-/Nullfactory.SSRSExtensions) provided is only a template and should be extended or re-implemented to work with your own requirement.
* The custom assembly should be deployed into the Global Assembly Cache, I suggest including as [part of the SharePoint Solution Package](https://msdn.microsoft.com/en-us/library/ee231595.aspx).
* Although this implementation was tested only with a Microsoft SQL Server data source type, I believe it would be possible to alter it to work with different connection types.

I would finally like to highlight the following articles as they were very informative and gave me a lot of insight into Code Access Security (CAS) and understand the security requirements I needed to address. They're a must read.

- [Simple-Talk - Code Access Security in ASP.NET 4.0](https://www.simple-talk.com/dotnet/.net-framework/code-access-security-in-asp.net-4.0/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part I](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-i/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part 2](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-2/)

## References
- [Technet - Features Supported by Reporting Services in SharePoint Integrated Mode](https://technet.microsoft.com/en-us/library/bb326290%28v=sql.105%29.aspx)
- https://social.msdn.microsoft.com/Forums/sqlserver/en-US/d4588e88-5cb9-47cf-817b-a69942c507ac/how-to-use-microsoftsharepointdll-api-in-custom-assembly-of-ssrs-report?forum=sqlreportingservices
- https://msdn.microsoft.com/en-us/library/ms154658.aspx

- https://msdn.microsoft.com/en-us/library/ms153561.aspx
- https://msdn.microsoft.com/en-us/library/ms154645.aspx
- https://msdn.microsoft.com/en-us/library/ms155034.aspx
- https://msdn.microsoft.com/en-us/library/ms153587.aspx
- https://msdn.microsoft.com/en-us/library/ms154507.aspx
- https://msdn.microsoft.com/en-us/library/ms152801.aspx
- https://msdn.microsoft.com/en-us/library/ms153693.aspx
- [Simple-Talk - Code Access Security in ASP.NET 4.0](https://www.simple-talk.com/dotnet/.net-framework/code-access-security-in-asp.net-4.0/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part I](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-i/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part 2](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-2/)

- https://msdn.microsoft.com/en-us/library/office/dn268593.aspx
- https://msdn.microsoft.com/en-us/library/ee231595.aspx