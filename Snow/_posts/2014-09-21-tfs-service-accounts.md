---
layout: post
title: Team Foundation Server - Service Accounts
category: ALM, Team Foundation Server
---
Application Lifecycle Management is an area that I have been wanting to improve for a while now. And what better way to do it than getting my self certified in the Microsoft ALM exams. I have decided to create a series of posts dedicated to ALM and sort of my "Road to Certification" posts.

In this post, I provide an overview of the different accounts that are required for the installation and smooth operation of TFS. 

## Overview

I created this mind map that graphically illustrates the accounts and required permissions by the different services in order to help me remember all the dependencies.

[![TFS Accounts Overview](/images/posts/TFSServiceAccounts/0_TFSAccountsOveriew.png)](/images/posts/TFSServiceAccounts/0_TFSAccountsOveriew.png)

While the majority of production installations would be deployed in a multi-tiered environment, I would normally assume that all of the accounts listed above are part of the same domain (with the exception of `DEPLOY` account. I will discuss cross domain considerations in a different post).

While I will go into detail in future posts, but for the initial installation the following accounts are required:

- `TFSInstall` - An installation account that would automatically be added as a TFS Administrator
- `TFSService` - Used to run the Team Foundation Service
- `TFSReports` - Used to run the SQL Server Reporting Service (SSRS)

<!--excerpt-->


###Log on as a Service
As shown in the mind map, several of the accounts need to be given the right to log on as a service on their appropriate machines. These steps would be needed to accomplish that:

1. Log onto the desired server.
2. Open the local group policy by executing `gpedit.msc` via a console.

	![Run Dialog](/images/posts/TFSServiceAccounts/1_RunDialog.png)

3. Navigate to the following tree node: `Computer Configuration > Security Settings > User Rights Assigment`

	![Local Group Policy](/images/posts/TFSServiceAccounts/2_gpedit.png)

4. Edit the `Log on as a service` entry by double clicking it and add the necessary accounts.

	![Local Group Policy](/images/posts/TFSServiceAccounts/3_AddLogOnAsAService.png)

###Log on Locally

The steps are very similar to Log on as a service. Follow steps 1-3 and on the last step choose `Allow log on locally` and add the accounts as necessary.

###SQL Server Reporting Services Permissions

Two service accounts need to be updated with permissions in order to properly integrate with SSRS. Here is a summary of what is required:

- `TFSReports` - Used as the service account for the reporting services. 
	- Allowed to log on locally on the Application Tiers as well as the report Server itself.
	- `TFSWareHouseDataReader` role on the report server.
- `TFSService` - This account needs to be added as a `Content Manager` in the report server.

I will walk through the process of installation and configuring in a future posts.

##References

- [MSDN - Service Accounts and Dependencies in Team Foundation Server](http://msdn.microsoft.com/en-us/library/ms253149.aspx)
- [MSDN - Grant permission to view or create SSRS reports in TFS](http://msdn.microsoft.com/en-us/library/bb737953(v=vs.110).aspx)
- [Technet - Add the Log on as a service right to an account](http://technet.microsoft.com/en-us/library/cc739424(v=ws.10).aspx)
- [Technet - Allow log on locally](http://technet.microsoft.com/en-us/library/cc756809(v=ws.10).aspx