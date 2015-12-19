---
layout: post
title: Strategy for Updating a SQL Server Database Schema via SSDT Delta Script
category: SQL Server, SQL Server Data Tools
---

This posts talks about the high level steps I went through in order to get the SQL Server related components ready for automation in a project I worked on recently. 

This project uses SQL Server Data Tools (SSDT) project in order to maintain the database schema in source control. Its output - the Data-tier Application Component Packages (DACPAC) gets deployed into the appropriate target environment via a WebDeploy package. And considering that the solution was designed as an Entity Framework (EF) database first approach, code first migrations were not a viable upgrade strategy.

Here are the steps I followed in order to bring the production environment up-to-date: 

<!--excerpt-->

1. Create a baseline DACPAC and move it into source control - this represents the schema currently in production.
2. Next, ensure that every time the SSDT project is built a post event would generate a differential delta script between the baseline and latest DACPAC. I tried to simplify the following command by wrapping it up within a powershell script:

		&"C:\Program Files (x86)\Microsoft SQL Server\110\DAC\bin\sqlpackage" /a:Script /sf:$SourceDacpac  /tf:$TargetDacpac /op:$OutputDeltaFile /tdn:$DBName /p:IncludeTransactionalScripts=True /p:IncludeCompositeObjects=True /p:ScriptDatabaseOptions=False /p:BlockOnPossibleDataLoss=True /v:TenantSchemaName=dbo

	Note that one of the parameters (`p:IncludeTransactionalScripts=True`) was to ensure that the script would be generated as a transaction.  

3. (optional) Perform any post processing on the generated delta script - In my specific use-case I had to tinker the script to work within a multi-tenant scenario. 
4. Deploy the generated delta script against the target environment. This can be done using a tools `SqlCmd` or a custom tool such as [https://github.com/rusanu/DbUtilSqlCmd](https://github.com/rusanu/DbUtilSqlCmd)
5. Upon successful release, update the baseline to the latest DACPAC file.

## References

- [Stack Overflow - ssdt - SQLPackage with Script Action does not produce any Copy Always scripts](http://stackoverflow.com/questions/22352298/sqlpackage-with-script-action-does-not-produce-any-copy-always-scripts) 
- [Stack Overflow - ssdt - What is the syntax for adding multiple arguments onto the "Variables" parameter in sqlpackage.exe?](http://stackoverflow.com/questions/15502659/what-is-the-syntax-for-adding-multiple-arguments-onto-the-variables-parameter) 
- [MSDN - SqlPackage.exe](https://msdn.microsoft.com/en-us/library/hh550080(v=vs.103).aspx)
- [Extract DacPac Using Command Line | phoebix](http://phoebix.com/2013/09/19/extract-dacpac-using-command-line/)
- [rusanu/DbUtilSqlCmd · GitHub](https://github.com/rusanu/DbUtilSqlCmd)