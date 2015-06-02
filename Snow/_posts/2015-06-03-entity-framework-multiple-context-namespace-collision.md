---
layout: post
title: Entity Framework Namespace Collisions When Working with Multiple Contexts
category: Entity Framework, SQL Server
---
I came across the following exception whilst attempting working with a solution that contained a *couple of* Entity Framework (EF) 6 database contexts.

	System.Data.Entity.Core.MetadataException

	Schema specified is not valid. Errors: 
    The mapping of CLR type to EDM type is ambiguous because multiple CLR types match the EDM type 'Setting'. Previously found CLR type 'SqlHelper.Primary.Setting', newly found CLR type 'SqlHelper.Secondary.Setting'.

    at System.Data.Entity.Core.Metadata.Edm.ObjectItemCollection.LoadAssemblyFromCache(Assembly assembly, Boolean loadReferencedAssemblies, EdmItemCollection edmItemCollection, Action`1 logLoadMessage)
    at System.Data.Entity.Core.Metadata.Edm.ObjectItemCollection.ExplicitLoadFromAssembly(Assembly assembly, EdmItemCollection edmItemCollection, Action`1 logLoadMessage)
    at System.Data.Entity.Core.Metadata.Edm.MetadataWorkspace.ExplicitLoadFromAssembly(Assembly assembly, ObjectItemCollection collection, Action`1 logLoadMessage)
    at System.Data.Entity.Core.Metadata.Edm.MetadataWorkspace.LoadFromAssembly(Assembly assembly, Action`1 logLoadMessage)
    at System.Data.Entity.Core.Metadata.Edm.MetadataWorkspace.LoadFromAssembly(Assembly assembly)
    at System.Data.Entity.Internal.InternalContext.TryUpdateEntitySetMappingsForType(Type entityType)
    at System.Data.Entity.Internal.InternalContext.UpdateEntitySetMappingsForType(Type entityType)
    at System.Data.Entity.Internal.InternalContext.GetEntitySetAndBaseTypeForType(Type entityType)
    at System.Data.Entity.Internal.Linq.InternalSet`1.Initialize()
    at System.Data.Entity.Internal.Linq.InternalSet`1.get_InternalContext()
    at System.Data.Entity.Internal.Linq.InternalSet`1.ActOnSet(Action action, EntityState newState, Object entity, String methodName)
    at System.Data.Entity.Internal.Linq.InternalSet`1.Add(Object entity)
    at System.Data.Entity.DbSet`1.Add(TEntity entity)
    at MultiContextConsoleApp.Program.Main(String[] args) in e:\shane\Projects\Orca\Sandbox\MultiContextConsoleApp\MultiContextConsoleApp\Program.cs:line 16
    at System.AppDomain._nExecuteAssembly(RuntimeAssembly assembly, String[] args)
    at System.AppDomain.ExecuteAssembly(String assemblyFile, Evidence assemblySecurity, String[] args)
    at Microsoft.VisualStudio.HostingProcess.HostProc.RunUsersAssembly()
    at System.Threading.ThreadHelper.ThreadStart_Context(Object state)
    at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
    at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
    at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)
    at System.Threading.ThreadHelper.ThreadStart()

<!--excerpt-->

## Reproducing the Problem

Here's a quick run down of the steps required to reproduce the scenario:
	
1. Start off by creating two databases.
2. Create a new table with the same name on each of them.

	![Database Schema](/images/posts/EFNamespaceConflict/10_DatabaseSchema.png)

2. Next, open up Visual Studio and create a Class Library project to host both EF database contexts.
3. Within the same project, create two EF (*ADO.NET Entity Data Model*) contexts; one for each database. Ensure that each context is under its own namespace.

	![Project Structure](/images/posts/EFNamespaceConflict/15_InitialProjectStructure.png)

	![Primary Context](/images/posts/EFNamespaceConflict/20_PrimaryContext.png)

	![Secondary Context](/images/posts/EFNamespaceConflict/30_SecondaryContext.png)

	![Primary Context Settings File](/images/posts/EFNamespaceConflict/40_PrimarySetting.png)

	![Secondary Context Settings File](/images/posts/EFNamespaceConflict/50_SecondarySetting.png)

4. Next, let's create a simple console application that would act as the client and invoke operations on the contexts. 
5. Run the application - an exception is thrown the moment there is any kind of interaction with either of the contexts. 

	![SSMS Context Menu](/images/posts/EFNamespaceConflict/60_Exception.png)

## The Solution

This issue is only reproducible when working with multiple database contexts that have tables with the same name and share the same assembly. It does not even matter that the contexts are across different namespaces. 

The problem appears to be how Entity Framework resolves namespaces, [this ticket goes into more detail](http://entityframework.codeplex.com/workitem/483). I was able to reproduce this with the latest version of EF and a fix does not appear to be on the immediate schedule. 

Luckily there are couple of workarounds which are pretty straightforward:

1. Make sure that that both contexts don't share tables with the same name - *not the most practical approach*.
2. Restructure the solution by isolating the database contexts within their own assemblies.

	![SSMS Context Menu](/images/posts/EFNamespaceConflict/70_NewProjectStructure.png)   

## References
- [CodePlex - Entity Framework - View Issue #483: Can't map two classes with same name from different namespace](http://entityframework.codeplex.com/workitem/483)
- [StackOverflow.com - c# - The mapping of CLR type to EDM type is ambiguous with EF 6 & 5?](http://stackoverflow.com/questions/14927391/the-mapping-of-clr-type-to-edm-type-is-ambiguous-with-ef-6-5)
- [MSDN Forum - EF4 Mapping of CLR type to EDM type is ambiguous error.](https://social.msdn.microsoft.com/Forums/en-US/5a8ea003-c6bc-4fc6-ad2a-634f09447c49/ef4-mapping-of-clr-type-to-edm-type-is-ambiguous-error?forum=adodotnetentityframework)