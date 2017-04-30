---
layout: post
title: Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 1 - Setup
category: Octopus Deploy, Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, ALM, Git 
---
I have seen [Octopus Deploy (OD)](https://octopus.com/) being mentioned frequently in the community but haven't had a chance to play around with it myself. Therefore, I think that attempting to deploy a Dynamics 365/CRM solution using Octopus Deploy seemed like the perfect exercise to learn more about the platform and its features.

I was curious if there were other developers who have released Dynamics CRM using Octopus Deploy and sure enough there was some one who had [already thought of that](https://www.youtube.com/watch?v=--9u5azwSb4).

In this series of posts I try to cover everything from setting up integration with Visual Studio Team Services(VSTS), updating a build definition to automatically publish itself and then finally setting up a simple release and deployment pipeline.

Related posts from the series:

- Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 1 - Setup 
- [Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 2 - Build](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-build-part-2/)
- [Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 3 - Release and Deployment](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-release-part-3/)

## Prerequisites

- Project structure and deployment scripts based on [generator-nullfactory-xrm](https://www.npmjs.com/package/generator-nullfactory-xrm) (with a minimum version of at least 1.4.0). [Read More](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/)
- The project checked into Visual Studio Team Services (VSTS) using either `git` or `tfsvc` with a working team build already setup. [Read More](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2).
- Installation of Octopus Deploy and ensure sure that it is publicly accessible. [Read More](https://www.youtube.com/watch?v=T0BQvMDXPNQ).

## The Strategy

When I saw the integration tasks provided by the OD extensions for VSTS, my first reaction was to divide the tasks between the build and the release in VSTS. I wanted to the the build definition to build and package the solution and pass it on to release management. Release management would then publishing the release-artifact to OD and explicitly remote trigger the OD deployment.

While it was certainly possible to set this up, its did not make sense to have two deployment services for the same pipeline. Therefore the strategy is going to be very similar to the one that I had done previously - VSTS would continue to act as the version control and build server but this time Octopus deploy would act as the release component.

The build server would handle creating compatible package and uploading it to the OD package repository. And OD takes over the management of the environment, release and deployment complexities.

![Release Strategy Overview](/images/posts/OctoDeployPt1/05_overview.png)

<!--excerpt-->

## Integrate Octopus Deploy with VSTS

Let's start of by integrating OD with VSTS.

1. Log into the Octopus Deploy admin panel with an user account that has enough permission to generate an API key.

	![API Profile](/images/posts/OctoDeployPt1/10_od_api_profile.png)

1. Click on the `Profile` and navigate to the `API keys` tab and generate a new API key.

	![Generate New API Key](/images/posts/OctoDeployPt1/20_generate_new_api_key.png)

1. Copy the newly generated API key. 

	![API Key](/images/posts/OctoDeployPt1/30_api_key.png)

1. Next, log into VSTS using the project collection administrator account.
1. Install the [Octopus Deploy Integration](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks) extension.

	![OD VSTS Integration](/images/posts/OctoDeployPt1/40_od_integ.png)

	![Integration Installation Complete](/images/posts/OctoDeployPt1/50_od_integ_install_complete.png)

1. Next, we create a OD connected service that would expose the build tasks required by the team build. Let's start off by navigating to the `Settings > Services` tab.
1. Clicking on the `New Service Endpoint` under the `Endpoints` tab.
1. Select `Octopus Deploy` from the list.

![Project structure](/images/posts/OctoDeployPt1/60_service_endpoint.png)

1. Provide the details of your OD environment and previously generated API key.

![Project structure](/images/posts/OctoDeployPt1/70_service_endpoint2.png)

Now that the VSTS environment is connected with OD, we move onto working on the build definition. In my [next post](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-build-part-2/) I will go about updating a previously setup build definition to automatically package the solution and publish it to Octopus Deploy. 

## References

- [July 2016 Meeting - Automated Deployment in CRM Using Octopus Deploy - YouTube](https://www.youtube.com/watch?v=--9u5azwSb4)
- [Get extensions for Visual Studio Team Services | Visual Studio Team Services](https://www.visualstudio.com/en-us/docs/marketplace/get-vsts-extensions)
- [Using the Team Foundation Build Custom Tasks - Octopus Deploy](https://octopus.com/docs/guides/use-the-team-foundation-build-custom-task)
- [How to create an API key - Octopus Deploy](https://octopus.com/docs/how-to/how-to-create-an-api-key)
- [Octopus 3.0 part 1: Introduction - YouTube](https://www.youtube.com/watch?v=FOxqnle2bCc)
- [Octopus 3.0 part 2: Installing the Octopus Deploy Server - YouTube](https://www.youtube.com/watch?v=T0BQvMDXPNQ)