---
layout: post
title: Setting up a Private Build Agent in Visual Studio Team Services - Part 2
category: Visual Studio Team Services, Team Build, ALM, Azure
published: draft
---



## Setting up the Scheduler

Now that we've provisioned our virtual machine and configured on it, let's go ahead and schedule it be active only during working hours.


1. Click the `Create` button and let the automation account provision.
2. Navigate to the run book via the resource group.
3. Click on the `Runbooks` widget to expand the blade.
4. The runbook gallery has a couple of really good graphical runbooks that suits my purpose - the `Start Azure V2 VMs` and `Stop Azure V2 VMs`. 
5. Click on the `Browse Gallery` button and select the `Start Azure V2 VMs` runbook. Inspect the flow and click the `Import` button when ready.
6. Confirm the name and click the `OK` button. This adds the runbook to the automation account.
7. Navigate into the newly added runbook and click the `Edit` button. I was happy with the out of the box script, so I hit the `Publish` button.
9. Once back on the runbook blade, click on `Schedules` widget and then click on the `Add a schedule` button.
10. Click on the `Link a schedule to your runbook` and then create a new schedule. Since this is the start script, provide the time at which you would want the VMs to start.
11. Click the `create` button once done.
12. 

## Alternate Approach