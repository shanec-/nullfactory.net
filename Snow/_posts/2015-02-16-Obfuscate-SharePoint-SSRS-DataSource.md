---
layout: post
title: Obfuscating a SharePoint-Integrated SSRS DataSource's Connection String  
category: SharePoint, SQL Server Reporting Services, Code Access Security
---

When SQL Server Reporting Services (SSRS) is deployed as an SharePoint integrated solution, it enables much of its functionality to be managed right from within SharePoint. Starting from the 2013 version, the integration between SharePoint and SQL Server Reporting Services 2012 is more tightly coupled than previous iterations. 

One feature in integrated mode is the ability to have the data sources (.rsds) and report files (.rdl) within a document library itself. This means that reports can reference a DataSource within any document library in the SharePoint site. 

In order for the report to work the user should have read permission on both the data source as well as the report file. The problem with this is that the same user can now potentially view the settings within the data source file, including the connection string.

## The Solution

In order to protect the connection string, I came up with a solution to obscure it through encryption. The solution can be broken down to two major steps:

1. Force the reports to get the connection string by evaluating an expression embedded within itself. 
2. Within this expression, call some custom code which manages the retrieval and decryption of the connection string. 

One of the limitations with this method is that you can no longer use a shared data source and each report has to have its credentials embedded.

In my example below, I will be retrieving the configuration string from a configuration list stored in the same SharePoint server. 

<!--excerpt-->

### Create the report extensions assembly

Here's a summary of steps used to create report extension. You can find a link to the full source at the bottom of the post.

1. Create new a class library to host your custom code.
1. Next, sign the assembly as we would be deploying it into the Global Assembly Cache. 
1. In order for the report server to call the custom code, the strong-named assembly must be marked with the `[assembly: AllowPartiallyTrustedCallers]` attribute. Let's do this now.

	![Allow Partially Trusted Callers](/images/posts/ObfuscateSSRS/10_AllowPartiallyTrustedCallers.png)

1. Create a new public class and method that would retrieve the connection string. This method would take the report server url as its only parameter. This parameter would be parsed and later used to generate the REST call to SharePoint. Next, add a `[SecuritySafeCritical]` attribute to the method, we need to this to perform operations that require access outside of the sandbox. 

	![GetConnectionStringMethod](/images/posts/ObfuscateSSRS/20_CodeGetConnectionString.png)

1. Now, create a helper method that retrieves the default credentials. This too would be decorated with the `[SecuritySafeCritical]` attribute. In the body of the method, I explicitly assert the `EnvironmentPermission(EnvironmentPermissionAccess.Read, "USERNAME")` permissions before using the `CredentialStore`. We would run into a security exception if the explicit assertion is not done. 

	![GetSecurityCredential Method](/images/posts/ObfuscateSSRS/50_CodeGetSecurityCredentials.png)

1. Finally, create the method that makes the REST call to retrieve the obfuscated connection string from a SharePoint configuration list.

	![GetConnectionStringFromSharePointList Method](/images/posts/ObfuscateSSRS/40_CodeGetWebPermission.png)

1. The `EnvironmentPermission` assertion operation has to be in its own method with its own `[SecuritySafeCritical]` attribute for it to work together with the `WebPermission` assertion. Otherwise the following exception would be thrown:

		Exception: System.Security.SecurityException Exception Message: Stack walk modifier must be reverted before another modification of the same type can be performed. Stacktrace:    at System.Security.CodeAccessSecurityEngine.Assert(CodeAccessPermission cap, StackCrawlMark& stackMark)
		   at System.Security.CodeAccessPermission.Assert()

	After numerous failed attempts at accessing the SPList via the SharePoint object model (I kept running into exceptions stating that I required full trust), I find that partial trust mode is [no longer supported and has been deprecated](https://msdn.microsoft.com/en-us/library/office/dn268593.aspx). As I was unable to find anyone online who had successfully got it to work, I opted to access the list via the REST service for its lesser dependencies and security permission requirements. 

1. Compile and deploy the assembly into the Global Assembly Cache.

### Setting up the Report

1. Open up a copy of your report using the [latest version of Report Builder](http://www.microsoft.com/en-us/download/details.aspx?id=29072). 
1. Right click on the work space area outside to bring up the context menu. Select the `Report Properties...` menu item which opens up the `Report Properties` dialog.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/60_ReportPropertiesContextMenu.png)

	![Build Process Parameter](/images/posts/ObfuscateSSRS/70_ReoprtPropertiesDialog.png)

1. Navigate to the `References` tab and add an entry for our custom assembly.
1. Next, create a new entry for the class containing our custom logic. Ensure that the class name is referred to using it full namespace and provide an instance name that would be used to call our code from within the report.  

	![Build Process Parameter](/images/posts/ObfuscateSSRS/80_ReportPropertiesDialog.png)

1. Now that our declarations have been done, edit the report data source for the report.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/90_EditDataSource.png)

1. Click on the `fx` button to open up the expressions dialog.
	
	![Build Process Parameter](/images/posts/ObfuscateSSRS/100_ConnStringSetExpression0.png)

1. Edit the expression to use the new custom method created previously. Pass in `Globals!ReportServerUrl` as the method parameter.
	
	![Build Process Parameter](/images/posts/ObfuscateSSRS/110_ConnStringSetExpression1.png)

1. Now, upload the updated report back into the SharePoint list hosting our report files. If you already have a report with the same name and with a shared data source linked to it, our connection details would be overwritten. In order to avoid this, delete the report in the list and re-upload our new one. 

1. Next, edit the Report Data Source by selecting the `Manage Data Sources` menu item in the context menu. Ensure that connection type is set to `Custom data source` and connection string to `Use connection string expression defined in the report`.

	![Build Process Parameter](/images/posts/ObfuscateSSRS/130_ManageDataSource.png)

	![Build Process Parameter](/images/posts/ObfuscateSSRS/120_ConnStringNotInitialized.png)

1. For the credentials, I will be using stored credentials. Note that the `Test Connection` button does not work if the connection string needs to be evaluated at runtime.

## Final Thoughts

* You can find the [source here](https://github.com/shanec-/Nullfactory.SSRSExtensions). It is meant only to be a template and should be extended to work with your own requirements.
* Since the custom assembly needs to be deployed into the Global Assembly Cache, I suggest including it as [part of the SharePoint Solution Package](https://msdn.microsoft.com/en-us/library/ee231595.aspx) when automating your deployment process. 
* Although this implementation was tested with a Microsoft SQL Server data source type, I suspect it can be altered to work with different connection types.

I would finally like to highlight the following articles as they gave me a lot of insight into Code Access Security (CAS) and better understanding of the security requirements I needed to address:

- [Simple-Talk - Code Access Security in ASP.NET 4.0](https://www.simple-talk.com/dotnet/.net-framework/code-access-security-in-asp.net-4.0/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part I](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-i/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part 2](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-2/)

## References
- [Technet - Features Supported by Reporting Services in SharePoint Integrated Mode](https://technet.microsoft.com/en-us/library/bb326290%28v=sql.105%29.aspx)
- [MSDN Forums - How to use "Microsoft.SharePoint.dll" API in custom assembly of SSRS report](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/d4588e88-5cb9-47cf-817b-a69942c507ac/how-to-use-microsoftsharepointdll-api-in-custom-assembly-of-ssrs-report?forum=sqlreportingservices)
- [MSDN - Code Access Security in Reporting Services](https://msdn.microsoft.com/en-us/library/ms154658.aspx)
- [MSDN - Using Custom Assemblies with Reports](https://msdn.microsoft.com/en-us/library/ms153561.aspx)
- [MSDN - Referencing Assemblies in an RDL File](https://msdn.microsoft.com/en-us/library/ms154645.aspx)
- [MSDN - Deploying a Custom Assembly](https://msdn.microsoft.com/en-us/library/ms155034.aspx)
- [MSDN - Asserting Permissions in Custom Assemblies](https://msdn.microsoft.com/en-us/library/ms153587.aspx)
- [MSDN - Accessing Custom Assemblies Through Expressions](https://msdn.microsoft.com/en-us/library/ms154507.aspx)
- [MSDN - How to: Debug Custom Assemblies](https://msdn.microsoft.com/en-us/library/ms153693.aspx)
- [Simple-Talk - Code Access Security in ASP.NET 4.0](https://www.simple-talk.com/dotnet/.net-framework/code-access-security-in-asp.net-4.0/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part I](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-i/)
- [Simple-Talk - What's New in Code Access Security in .NET Framework 4.0 - Part 2](https://www.simple-talk.com/dotnet/.net-framework/whats-new-in-code-access-security-in-.net-framework-4.0---part-2/)
- [MSDN - Deciding between apps for SharePoint and SharePoint solution - Partial-trust user code is deprecated](https://msdn.microsoft.com/en-us/library/office/dn268593.aspx)
- [MSDN - How to: Add and Remove Additional Assemblies](https://msdn.microsoft.com/en-us/library/ee231595.aspx)