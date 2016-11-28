---
layout: post
title: Scheduling the Uptime of an Azure Virtual Machine 
category: Azure
---
In a previous post I setup a [build agent in a private pipeline](/2016/11/setting-up-a-private-build-agent-in-visual-studio-team-services/) - hosted within a Azure Virtual Machine (VM). In this one, I try to minimize the uptime of the VM by making it run only during work hours - I require it to start automatically first thing in the morning and then turn itself off at the close of business. In order to achieve this I make use of the [Azure Automation](https://azure.microsoft.com/en-au/services/automation/) feature.


1. Let's start off by logging into our Azure account and creating a new `Automation Account` entry. 

	![Search Automation Account](/images/posts/AzureVmSchedule/10_Automation.png)

	![Create Automation Account](/images/posts/AzureVmSchedule/20_CreateAutomation.png)

1. Provide the required details:

	 - `Name` - Name for the automation account.
	 - `Subscription` - Azure subscription for this account.
	 - `Resource Group` - Create or select an existing resource group from the list. 
	 - `Location` - Select an available region from the list.
	 - `Create Azure Run As account` - Select `Yes`.

	![Add Automation Account](/images/posts/AzureVmSchedule/30_AddAutomationInfo.png)

1. Click the `Create` button to start provisioning the automation account.
1. Once done, navigate to it's blade via the resource groups and click on the `Runbooks` widget.

	![Create runbook](/images/posts/AzureVmSchedule/40_CreateRunbook.png)
<!--excerpt-->
1. The runbook gallery had a couple of really good graphical runbooks that suited my purposes - the `Start Azure V2 VMs` and `Stop Azure V2 VMs`. 

	Click on the `Browse Gallery` button, select the `Start Azure V2 VMs` graphical runbook and then click `OK` to confirm the selection.

	![Choose Graphical Runbook](/images/posts/AzureVmSchedule/50_ChooseRecipe.png)

1. Inspect the flow and click the `Import` button when ready.
	
	![Import Graphical Runbook](/images/posts/AzureVmSchedule/60_ImportRecipe.png)

1. Navigate into the newly added runbook and click the `Edit` button. Make any alterations if required and click the `Publish` button.

	![Publish Runbook](/images/posts/AzureVmSchedule/70_PublishRunbook.png)

1. Back on the runbook blade, click on `Schedules` widget and then click on the `Add a schedule` button.

	![Add Schedule](/images/posts/AzureVmSchedule/80_AddSchedule.png)

1. Click on the `Link a schedule to your runbook` and then create a new schedule. Provide the time and recurrence for the script execution. [Read more about scheduling here](https://docs.microsoft.com/en-au/azure/automation/automation-scheduling-a-runbook).

	![Finalize Schedule](/images/posts/AzureVmSchedule/90_FinalizeSchedule.png)

1. Next, select `Configure parameters and run settings` and provide a virtual machine name or resource group which would start on execution.

	![Schedule parameters](/images/posts/AzureVmSchedule/100_Parameters.png)

1. Click the `ok` button once done. You should get a confirmation that the schedule is linked to a runbook. 

	That's it. The runbook would execute at the pre-defined time.

1. Repeat the above steps again to create a stop script using the `Stop Azure V2 VMs` graphical runbook. 

## References

- [Automation – Cloud process & workflow automation | Microsoft Azure](https://azure.microsoft.com/en-au/services/automation/)
- [Scheduling a runbook in Azure Automation | Microsoft Docs](https://docs.microsoft.com/en-au/azure/automation/automation-scheduling-a-runbook)