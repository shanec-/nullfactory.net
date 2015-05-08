---
layout: post
title: Parameterize Schema Name within a SSDT Database Project for Multi-Tenant Solutions
category: SQL Server Data Tools
---

I've been working on an multi-tenant solution recently and have been trying to come up with an efficient way to manage the database deployment and upgrade. The database is designed to segregate each tenant's data under its own schema namespace as such I need to generate a re-useable script that can be deployed against each tenant. The approach I am going to take is to first source control the database schema within a SQL Server Data Tools (SSDT) database project and then use it to generate the script that can be parameterized with the tenant information.

I first parameterized the the schema name as a SQLCMD variable - `$TenantName`: 

![SqlCmd Variable](/images/posts/DeploymentPlanModifer/10_varcmd.png)

Next I tried to replace the schema name with the new variable, but this did not work as trying to build the solution now returns with a 71502 error as the project is no longer able to resolve and validate schema objects.

![Schema Validation](/images/posts/DeploymentPlanModifer/20_SchemaValidation.png)
	
SQLCMD does not have any complaints if I replace the `[dbo].` with `[$TenantName]` in the generated script so its the SSDT project that is attempting to maintain the integrity of database. 

One possible way to overcome this is to [suppress the 71502](http://stackoverflow.com/questions/10826014/suppress-some-warnings-in-sql-server-ssdt) by turning them into [warnings](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/9b698de1-9f6d-4e51-8c73-93c57355e768/treat-specific-warning-as-error?forum=ssdt). The disadvantage in this approach is that you loose the rich validation in exchange for something that is essentially a deployment convenience.

Another duct tape and bubble gum approach would be to just have some kind of post deployment operation that does a find and replace on the schema name. Sure it would work, but that's not going to be reliable in the long run. 

A little bit of research reveals that the proper way to alter the creation of deployment script process is to create a deployment plan modifier. A deployment plan modifier is essentially a class that inherits `DeploymentPlanModifier` and allows you to inject custom actions when deploying a SQL project. There does not seem to be much formal documentation on the process, so I relied a lot on [this article in MSDN](https://msdn.microsoft.com/en-US/library/dn306642(v=vs.103).aspx), the sample [DACExtensions](https://github.com/Microsoft/DACExtensions) and what forum posts I could find. So with a lot of trial and error I wrote my own plan modifier that would replace the schema identifiers when the database project is published.

<!--excerpt-->

## How it works

There are two main components to the solution; first one is the `SchemaSubstituteScriptContributor` that is based off of `DeploymentPlanModifier` that hosts and coordinates the injection process. And the other is `OverrideSchemaVisitor` which is based off of `TSqlFragmentVisitor` and does the actual schema substitution.

On the `SchemaSubstituteScriptContributor` class, I've overridden the `OnExecute` method to look for steps of type `DeploymentScriptDomStep` and once it find it, navigate down the class hierarchy until it reaches the actual `TSqlStatement`. And into this `TSqlStatement` I pass in a new instance of `OverrideSchemaVisitor` via the `Accept` method.

The `OverrideSchemaVisitor` has overridden methods to handle each type of statement that have schema references and need to be altered.

## Installation and Usage

Visual Studio loads up any extensions that are available in the extensions folder every time it starts up. 
Now copy the assembly to the extensions folder that Visual Studio checks when it starts-up. I had some trouble locating as it was not in the location that the article mentioned (see *troubleshooting*). Eventually I found out that mine was located at `%ProgramFiles(x86)%\Microsoft Visual Studio 12.0\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\120\Extensions`. 

![Extensions Folder](/images/posts/DeploymentPlanModifer/30_ExtensionsFolder.png)

Next, in order for the database project to make use of the new extensions, I add the following properties to the database project `SqlProj` file:
 
	  <PropertyGroup>
	    <DeploymentContributors>	
	      $(DeploymentContributors);Nullfactory.SchemaSubstitute
	    </DeploymentContributors>
	    <ContributorArguments Condition="'$(Configuration)' == 'Debug'">
	      $(ContributorArguments);Nullfactory.SchemaSubstitute.OldSchemaName=dbo;Nullfactory.SchemaSubstitute.NewSchemaName=$TenantSchema;
	    </ContributorArguments>
	  </PropertyGroup>

The parameters for the extension are passed via the `<ContributorArguments>` tag in a key-value pair format; where `Nullfactory.SchemaSubstitute.OldSchemaName` is the source schema name and `Nullfactory.SchemaSubstitute.NewSchemaName` is the new schema name. 

Now that everything is setup, anytime the project is published the schema name would be substituted appropriately. 

## Troubleshooting

My first problem was trying to figure out where to deploy the extensions. Although the source article stated that it should be in the `%Program Files%\Microsoft SQL Server\110\DAC\Bin\Extensions` folder, Visual Studio refused to recognize it.

![Unable to Load Extensions](/images/posts/DeploymentPlanModifer/40_UnableToLoadExtension.png)

Digging through the forums, I found out that the locations in which visual studio checks for extensions depends on how it was installed. The multiple locations are there in [order to avoid conflicts and maintain backward compatibility](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/be484b63-a6cc-4dac-a2c2-78a56ff5b502/where-is-the-microsoftsqlserverdacdll-that-includes-support-for-sql-server-2014?forum=ssdt). Luckily, I found another post that shows how to [definitively identify the location that is being checked](https://social.msdn.microsoft.com/Forums/en-US/eb0ccd5b-e7e0-4d90-950b-a4c696f0bd6e/deployment-contributor-could-not-be-loaded?forum=ssdt):

> 1. Open a new command prompt as Administrator.
> 2. Run the following command
> 
> 	`logman create trace -n DacFxDebug -p "Microsoft-SQLServerDataTools" 0x800 -o "%LOCALAPPDATA%\DacFxDebug.etl" -ets`
> 	`logman create trace -n SSDTDebug -p "Microsoft-SQLServerDataToolsVS" 0x800 -o "%LOCALAPPDATA%\SSDTDebug.etl" -ets`
> 
> 3. Run whatever the target/issue scenario is in SSDT.
> 4. Go back to the command prompt and run the following commands
> 
> 	`logman stop DacFxDebug -ets`
> 	`logman stop SSDTDebug -ets`
> 
> The resulting ETL files will be located at `%LOCALAPPDATA%\SSDTDebug.etl & %LOCALAPPDATA%\DacFxDebug.etl` and can be navigated to using Windows Explorer.
> The `DacFxDebug.etl` file will contain extension load information. This can be opened and analyzed using the Windows Event Viewer.
> To do this, open the Windows Event Viewer application. In the right-hand panel, select Open Saved Log. Navigate to the location where you saved the log, open, and review the contents of the trace.

![Event Viewer extensions location](/images/posts/DeploymentPlanModifer/50_EventViewerLocation.png)

I also had some trouble with referencing correct version of the assemblies. Here is the final list of references and the locations that they resided in. I am using SQL Server Data Tools version 12.0.50318.0 at the time of writing this post.

	System.ComponentModel.Composition - Framework

	Microsoft.Data.Tools.Schema.Sql.dll - `C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\120\Microsoft.Data.Tools.Schema.Sql.dll`
	Microsoft.SqlServer.Dac.dll - `C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\120\Microsoft.SqlServer.Dac.dll`
	Microsoft.SqlServer.Dac.Extensions.dll - `C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\120\Microsoft.SqlServer.Dac.Extensions.dll`
	Microsoft.SqlServer.TransactSql.ScriptDom.dll - `C:\Program Files (x86)\Microsoft SQL Server\120\SDK\Assemblies\Microsoft.SqlServer.TransactSql.ScriptDom.dll`

## Final thoughts

Its worth mentioning that this is not a problem for scripts that do not have a build action and scripts such as the pre and post deployment scripts are able to use the parameterized schema variable with no additional effort. 

Next step for me is to look at integrating this into and publishing script as part of a team build. Stay tuned.  

I've hosted my final [implementation here](https://github.com/shanec-/Nullfactory-DACExtensions) along with a sample database demonstrating its usage.

## References

- [MSDN - Walkthrough: Extend Database Project Deployment to Modify the Deployment Plan](https://msdn.microsoft.com/en-US/library/dn306642(v=vs.103).aspx)
- [MSDN - Walkthrough: Extend Database Project Build to Generate Model Statistics](https://msdn.microsoft.com/en-us/library/dn306080(v=vs.103).aspx)
- [Microsoft/DACExtensions](https://github.com/Microsoft/DACExtensions)
- [MSDN Forum - Deployment Contributor could not be loaded](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/eb0ccd5b-e7e0-4d90-950b-a4c696f0bd6e/deployment-contributor-could-not-be-loaded?forum=ssdt)
- [Technet - PackageOptions.ContributorArguments Property (Microsoft.SqlServer.Dac)](https://technet.microsoft.com/en-us/library/microsoft.sqlserver.dac.packageoptions.contributorarguments(v=sql.110).aspx)
- [Suppress some warnings in SQL Server SSDT - Stack Overflow](http://stackoverflow.com/questions/10826014/suppress-some-warnings-in-sql-server-ssdt)
- [MSDN Forum - Treat specific warning as error?](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/9b698de1-9f6d-4e51-8c73-93c57355e768/treat-specific-warning-as-error?forum=ssdt)
- [NuGet.Operations/Microsoft.SqlServer.Dac.Extensions.xml at master · NuGet/NuGet.Operations](https://github.com/NuGet/NuGet.Operations/blob/master/ext/SqlServer/Microsoft.SqlServer.Dac.Extensions.xml)