---
layout: post
title: TFS 2013 - Enabling Reporting, Warehouse and Analysis Services
category: Team Foundation Server
---

Suppose you already have an Team Foundation Server (TFS) environment where you have opted-out from configuring the Reporting Services and Analysis Services during the installation. The following steps would help you to configure the warehouse and reporting functionality.

## Pre-Requisites

1. Ensure that the SQL Server client tools are installed on the Application Tier.

	If this is not already done, you would most likely receive a `TF400465` error when attempting to edit the configuration. `TF400465` states that client tool are needed to be installed on the application tier in order for the analysis services to function properly.

	This can be fixed by re-running the SQL Server setup and adding the `Client Tools Connectivity` feature [[Read More]](http://msdn.microsoft.com/en-us/library/dd578652.aspx).

	Once installed, make sure to restart the TFS administration console.

1. Ensure that `Management Tools - Complete` is installed on at least one of the servers in your topology. This is because SQL Server Management Studio requires the complete version installed in order to manage Analysis Services. This step optional but makes it easy for troubleshooting or future maintenance.
1. Ensure that the Analysis Services have been installed and is up and running.
1. Ensure SQL Server Reporting Services is installed and configured in native mode [[Read More]](http://msdn.microsoft.com/en-us/library/aa545752%28v=cs.70%29.aspx).
1. This goes without saying, Analysis Services and Reporting Services are not available on the express version of SQL Server.

<!--excerpt-->

## Configuration

1. Navigate to TFS Administration Console `Application Tier > Reporting`. Notice that the services are not configured.

	![](/images/posts/TFSEnableReportingAnalysisServices/1_ConsoleUnconfigured.png)

1. Click `Edit` to edit the configuration.
1. Check the `Use Reporting` check box to get started.
1. Configure the warehouse house by first selecting the SQL server from the server list.

	![](/images/posts/TFSEnableReportingAnalysisServices/2_WarehouseDetails.png)

1. Next, provide a name for the warehouse database; I gave it `Tfs_Warehouse` as it is what the latest versions of TFS provides by default.
1. Click `Test Connection` to ensure the connectivity to the server.
1. Switch over to the `Analysis Services` tab and select the server for the analysis services from the server list.

	![](/images/posts/TFSEnableReportingAnalysisServices/3_AnalysisDetails.png)

1. Provide the database name `Tfs_Analysis`.
1. Provide the username and password for accessing the data sources.
1. Click `Test Connection` to ensure the connectivity to the server.
1. Next select the `Reports` tab.
1. Select the report server from the server list.

	![](/images/posts/TFSEnableReportingAnalysisServices/4_ReportingDetails.png)

1. Click the `Populate URLs`. This would retrieve the Web Service and Report Manager urls. In case they are not populated automatically, provide the details manually.
1. Provide the username and password for accessing the data sources.
1. Click `OK` to save the settings.
1. Click the `Start Jobs` to activate the services.

	![](/images/posts/TFSEnableReportingAnalysisServices/5_ConsoleConfigured.png)

1. The final step is to rebuild the warehouse. This would prepare the database by building the necessary schema and any pre-requisite operations. This can be achieved in one of the following ways:
	1. Executing the `RebuildWarehouse` command [[Read More]](http://msdn.microsoft.com/en-us/library/ee349264.aspx).
	2. Navigate to the `WarehouseControlService` webservice and execute `ProcessAnalysisDatabase` with `processingType` of `Full` [[Read More]](http://msdn.microsoft.com/en-us/library/ff400237.aspx).
	3. Open up the TFS Administration Console and navigate to `Application Tier > Reporting > Start Rebuild `

		![](/images/posts/TFSEnableReportingAnalysisServices/6_RebuildConfirmation.png)

## Verification

Once the `RebuildWarehouse` command has been executed, verify that the warehouse jobs are running. This can be done by logging onto the application tier and then navigating to the `WarehouseControlWebService` using the following url and executing the `GetProcessingStatus` action:

		http://localhost:8080/tfs/TeamFoundation/Administration/v3.0/WarehouseControlService.asmx

This would return a list of all the jobs that are being executed on the server. The first time I viewed the results, I was a bit overwhelmed by the number of errors shown. If you take a moment and study the errors you realize that these are warnings about SQL locks. Given enough time, and depending on the size of the Project Collections, the errors (warnings) should resolve themselves. As far as I can tell these warnings are expected in a full rebuild.

## References
* [MSDN - How to Configure Reporting Services](http://msdn.microsoft.com/en-us/library/aa545752%28v=cs.70%29.aspx)
* [MSDN - Manually install SQL Server for Team Foundation Server](http://msdn.microsoft.com/en-us/library/dd578652.aspx)
* [MSDN - RebuildWarehouse Command](http://msdn.microsoft.com/en-us/library/ee349264.aspx)
* [MSDN - Manually process the data warehouse and analysis services cube for Team Foundation Server](http://msdn.microsoft.com/en-us/library/ff400237.aspx)
