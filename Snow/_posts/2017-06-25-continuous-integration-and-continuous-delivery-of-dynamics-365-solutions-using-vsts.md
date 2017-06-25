---
layout: post
title: Continuous Integration and Continuous Delivery of Dynamics 365 Solutions using VSTS
category: ALM, Visual Studio Team Services, Dynamics CRM, Dynamics CRM Online, Git
---
In previous posts I've setup build and release defintions for Dynamics CRM solutions and in this one I try my attempt at setting up an automated Continuous Delivery pipeline. There are two popular strategies used to make sure that [every change made to a system can be deployed](https://puppet.com/blog/continuous-delivery-vs-continuous-deployment-what-s-diff) successfully to secondary environment:

- Continuous Integration (CI)

  A CI build is one that is usually kicked off any time a piece of code is checked-in in order to give early visibility of any potential failures. The success of a build is dependent on the actual build plus any unit tests.
- Scheduled

  A scheduled build, also commonly referred to as a `Nightly Build`, is a build and release that is performed on a regular cadence - usually every night. Once again, on the successful completion of the build and automated unit test, a new release is pushed to the staging environment.

Both methods appear to be doing the same thing, so when should I choose a CI build + release over a scheduled build? The most [common metric](https://stackoverflow.com/questions/417134/continuous-integration-vs-nightly-builds) appears to be the duration of your build and unit tests. The size of the team, the project and code complexity and the frequency of check-ins can have an impact on the time it takes for build and long running unit tests to execute. In these scenarios a `Scheduled` build makes more sense.

## Overview

I was disappointed to find out that, at the time of writing this post, VSTS release definitions does not provide an option to trigger off a successful a scheduled build. The only options available are to trigger it off of a CI build or the automated schedule on the release itself. 

Luckily, Rene van Osnabrugge's [excellent post](https://roadtoalm.com/2017/03/30/trigger-release-pipeline-only-for-scheduled-builds/) provides us with a workaround for this restriction. Here's an overview how it works:

- Setup a single build that performs all three roles (manually invoked, CI and scheduled). 
- Add a task into the build that would to figure out which method was used to invoke. 
- Apply the the reason as a build tag onto itself. 
- Update the release definition to filter the the release trigger to be filtered only the scheduled tags.

Lets get started setting it up.
<!--excerpt-->

## Prerequisites
- Visual Studio Team Services
- Existing team build - I will be re-using a Dynamics CRM project that I have used in [my previous post](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2/). 
- Existing release definition - Once again, I am using one that I've setup [previously](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/).

## The Build Definition

As in my previous posts I will be using a standard Dynamics CRM build for this demonstration. Follow the instructions in [my previous post](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2/) for setting this up .

![Build Definition](/images/posts/CrmCICD/10_BuildDefinition.png)

Now let's make the following modifications:

1. From within the build definition, navigate to the `Triggers` tab.
2. Enable the Continuous Integration trigger.
3. Check the `Batch changes while a build is in progress` check box. 
4. Add a new Path filter by clicking the `add` button.
5. Set the type to `Exclude` and `Path specification` to `/Nullfactory.Xrm.Tooling`. We don't really care for our tools to  This is so that we only trigger on the actual solution and not the supporting tools/project.
6. Now enable the scheduled trigger.
7. Provide a schedule that works for your team.
8. Finally, save the changes.

![Build Definition Trigger](/images/posts/CrmCICD/20_BuildDefinitionTrigger.png)

### Tagging The Build

Next, we update the build to tag itself upon a successful build.

1. I downloaded the [Set-BuildTagToTrigger.ps1](https://gist.githubusercontent.com/renevanosnabrugge/3fa094790fab2b06db970d0da81a71b0/raw/a99220346be3803297412a4b580d8e1bd6dcbcf0/Set-BuildTagToTrigger.ps1) script and added it part of the `Nullfactory.Xrm.Tooling/Scripts` project folder.
2. Commit and push the changes to the remote repository.
3. From within the build editor, click on the `Add Task` button and select a PowerShell task. I placed it as the last step in the order.
4. Configure it with the following values:

  - Display Name :`Tag Build Reason`
  - Type : `File Path`
  - Script Path : `Nullfactory.Xrm.Tooling/Scripts/Set-BuildTagToTrigger.ps1`
   - Arguments : `-BuildId $(Build.BuildId)`

	![Set Tag Reason PowerShell](/images/posts/CrmCICD/30_SetTagReasonPowerShell.png)

5. Ensure that the task only runs if all previous tasks have succeeded. Do this by navigating into `Control Options` section and selecting the option within the `Run this task` drop down.

6. In order for the the script to have authenticated access to the REST service it requires access to the OAuth token. Do this now navigating to the `Options` tab and enabling the `Allow scripts to access OAuth token`.

	![OData Token](/images/posts/CrmCICD/40_AllowODataToken.png)

7. Queue a new build and verify that a tag was associated with the build. Note that since we invoked the build explicitly, a `manual` tag was added.

	![Verify Build](/images/posts/CrmCICD/50_VerifyBuild.png)

The tags applied to the new build are the same values as the build reason environmental variable. We only care about the following scenarios:

- `Manual`
- `IndividualCI`
- `BatchedCI`
- `Schedule`

In fact, we only care about the `Schedule` typed of builds. Find the full list of available values [here](https://www.visualstudio.com/en-us/docs/build/define/variables).

## The Release Definition

Now that the build is complete and we've proven that the solution is good enough to be deployed, let's create and augment our release definition.

Follow the steps [described here](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-release-part-3/) to create the basic release definition. Ensure that the linked source is the build definition we created earlier.

1. From within the release definition edit screen, navigate to the `Triggers` menu.
2. Check the `Continuous Deployment` check box.
3. Select the build as the source from the drop down and the branch it should monitor.
4. Add `schedule` as the tag filter.
5. Click `Save` to confirm changes.

![Updated Release Definition](/images/posts/CrmCICD/60_ReleaseDefinition.png)

## Verification

1. Verify that the when the build is triggered via the schedule and that it has a `schedule` tag is associated upon completion.

  ![Verify Build](/images/posts/CrmCICD/70_ScheduledBuild.png)

2. Verify that a release was deployed off of it.

  ![Verify Build](/images/posts/CrmCICD/80_TriggeredRelease.png)

## References

- [What is Continuous Integration? | DevOps](https://www.visualstudio.com/learn/what-is-continuous-integration/)
- [What is Continuous Delivery? | DevOps](https://www.visualstudio.com/learn/what-is-continuous-delivery/)
- [Continuous Delivery Vs. Continuous Deployment: What's the Diff? | Puppet](https://puppet.com/blog/continuous-delivery-vs-continuous-deployment-what-s-diff)
- [Continuous Integration vs. Nightly Builds - Stack Overflow](https://stackoverflow.com/questions/417134/continuous-integration-vs-nightly-builds)
- [Build definition triggers](https://www.visualstudio.com/en-us/docs/build/define/triggers)
- [Trigger Release Pipeline only for Scheduled builds | The Road to ALM](https://roadtoalm.com/2017/03/30/trigger-release-pipeline-only-for-scheduled-builds/)
- [Use a PowerShell script to customize your build process](https://www.visualstudio.com/en-us/docs/build/scripts/)
- [Build variables](https://www.visualstudio.com/en-us/docs/build/define/variables)