---
layout: post
title: Generate a Clean Up Script to Drop All Objects in a SQL Server Database
category: SQL Server
---
I needed a quick and reusable way to drop all SQL server objects from an Azure database. The objective was to have some kind of process to clean up and prep the database before the main deployment is kicked off. And given that I am particularly biased towards using a sql script my search for a solution focused around it.

In addition to actually dropping the artifacts, the script should be aware of the order in which it should do it - that is to drop the most dependent objects first and work its way towards the least dependent ones. And my nice-to-have feature is to be able to parameterize the schema name so that it could be used with a multi-tenant database schema.

I saw a few possible solutions and finally settled on using the [out-of-the-box feature](http://stackoverflow.com/questions/536350/drop-all-the-tables-stored-procedures-triggers-constraints-and-all-the-depend) that's already available through SQL Server Management Studio (SSMS).

1. Open up SQL Server Management Studio.
2. Select `Task  > Generate Script...` on on your the database context menu. This would open up the `Generate and Publish Scripts` dialog.

	![SSMS Context Menu](/images/posts/GenerateDropScript/10_ContextMenu.png)
 
3. First, navigate to the `Choose Objects` tab and select all the objects that need to be dropped.
4. Next, on the `Set Scripting Options` tab, select the preferred output location.

	![Set Scripting Options](/images/posts/GenerateDropScript/20_SetScriptingOptions.png)
 
5. Next, click the `Advanced` button which result in the `Advanced Scripting Options` dialog.
	
	![Advanced Scripting Options](/images/posts/GenerateDropScript/30_AdvancedScriptingOptions.png)

6. Navigate down towards to and change the `General > Script DROP and CREATE` option to `Script DROP`.
7. Set the default values for the rest of the steps and finally click the `Finish` button.

<!--excerpt-->

SSMS sorts out the dependencies and generates a script similar to the one below. Note that the statements are in the required sequence.

    USE [test]
    GO
    ALTER TABLE [dbo].[Contact] DROP CONSTRAINT [FK_Contact_Company]
    GO
    /****** Object:  Table [dbo].[Contact]Script Date: 6/2/2015 9:33:36 AM ******/
    DROP TABLE [dbo].[Contact]
    GO
    /****** Object:  Table [dbo].[Company]Script Date: 6/2/2015 9:33:36 AM ******/
    DROP TABLE [dbo].[Company]
    GO

I finally made the following tweaks to convert the script to a `SQLCMD` script and parameterize the schema:

	:setvar TenantSchemaName "scm"
	
	ALTER TABLE [$(TenantSchemaName)].[Contact] DROP CONSTRAINT [FK_Contact_Company]
	GO
	/****** Object:  Table [$(TenantSchemaName)].[Contact]    Script Date: 6/2/2015 9:33:36 AM ******/
	DROP TABLE [$(TenantSchemaName)].[Contact]
	GO
	/****** Object:  Table [$(TenantSchemaName)].[Company]    Script Date: 6/2/2015 9:33:36 AM ******/
	DROP TABLE [$(TenantSchemaName)].[Company]
	GO

Although, I was not really focused on automation, it should not be too difficult to integrate it with existing automated processes.

## References

- [StackOverflow.com - sql server - Drop all the tables, stored procedures, triggers, constraints and all the dependencies in one sql statement - Stack Overflow](http://stackoverflow.com/questions/536350/drop-all-the-tables-stored-procedures-triggers-constraints-and-all-the-depend)