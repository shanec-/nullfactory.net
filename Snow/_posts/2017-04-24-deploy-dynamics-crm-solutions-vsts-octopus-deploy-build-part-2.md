---
layout: post
title: Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 2 - Build
category: Octopus Deploy, Dynamics CRM, Dynamics CRM Online, generator-nullfactory-xrm, Visual Studio Team Services, ALM, Git 
---
This is the second part of a series of posts where I attempt to deploy a Dynamics 365/CRM solution using Visual Studio Team Services (VSTS) and Octopus Deploy (OD). Previously, I setup an OD server and integrated it with VSTS - the steps describe in that post act as a prerequisite for this one. In this post we go about modifying an existing Dynamics CRM build definition to automatically package the release-artifact and upload it to the OD server for deployment. 

Related posts from the series:

- [Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 1 - Setup](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-setup-part-1/) 
- Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 2 - Build
- [Deploy Dynamics CRM Solutions using VSTS and Octopus Deploy - Part 3 - Release and Deployment](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-release-part-3/)

## Publishing Packages

There are a few different ways of getting a package into Octopus Deploy:

- Remotely upload a package to Octopus Deploy's built-in repository.
- Make octopus deploy pull packages from an external feed. Find more information [here](https://www.visualstudio.com/en-us/docs/package/overview) on setting up VSTS to publish an package feed.
- Manually upload the package into the built-in repository.

And the method of publishing the package has no bearing on the end deployment -I chose to explicitly push to the remote octopus deploy server. 

<!--excerpt-->

## Updating the Build Definition

A quick summary of the notable steps in the existing build definition:

![Existing Build Definition](/images/posts/OctoDeployPt2/10_starting_build_def.png)

1. The actual `msbuild` command that is being used to build the solution.
2. Copy the built artifacts to the staging folder.
3. Explicitly Copy Deployment Scripts into the staging folder.
4. Delete binaries (such as plugins and workflows) that have already been packaged as part of the solution package. As these serve no purpose in the final drop, I explicitly delete them to reduce the file size.
5. Publish the files in the staging folder into the final drop artifact.

Review my previous post on setting up the build definition [here](/2016/11/release-strategy-for-dynamics-crm-setting-up-the-build-part-2).

Now let's start implementing the changes to the definition:

1.  Add the `Package Application` task.

	![Add Package Application Task](/images/posts/OctoDeployPt2/20_add_package_app.png)

1. Configure the task by providing the following details:

	- `Package ID` : This is unique identification for the package. I provided `gnxdemo.crm`.
	- `Package Format` : Choose between NuGet and Zip. I chose NuGet. 
	- `Package Version` : Configure this as appropriate. The generated package version should follow a valid SemVer versioning strategy.
	- `Source Path` : Set this to the `$(build.artifactstagingdirectory)`
	- `Output Path` : Set it to a dedicated folder within the staging directory. I used `$(build.artifactstagingdirectory)\OctopusDeploy`

	![Configure Package Application Task](/images/posts/OctoDeployPt2/30_config_package_app.png)

1. Next, add the `Push Package` task.

	![Add Push Package Task](/images/posts/OctoDeployPt2/40_add_push_package.png)

1. Configure the task by providing the following details:

	- `Octopus Deploy Server` : Choose the available server.
	- `Package`: This is the location of the package file. Since I am only creating a single package I setup the path resolution using a wildcard `$(build.artifactstagingdirectory)\OctopusDeploy\*.nupkg`

	![Configure Push Package Task](/images/posts/OctoDeployPt2/50_config_package.png)

1. Remove the `Publish Artifact: drop` task as it is no longer necessary.

	![Remove Publish Artifact Task](/images/posts/OctoDeployPt2/60_remove_pushlish_artifacts.png)

1. Queue a new build to make sure that everything is in order.

	![Queue New Build](/images/posts/OctoDeployPt2/70_queued_build.png)

1. Log into the Octopus Deploy admin panel. Navigate to the `Library > Packages` and ensure that the package has successfully uploaded.

	![Available Packages](/images/posts/OctoDeployPt2/80_published_package.png)

That completes the changes to the build definition. In the [next post](/2017/04/deploy-dynamics-crm-solutions-vsts-octopus-deploy-release-part-3/) I will go about configuring the environments, project definition and the deployment process within Octopus Deploy.

## References

- [Using the Team Foundation Build Custom Tasks - Octopus Deploy](https://octopus.com/docs/guides/use-the-team-foundation-build-custom-task)
- [Package Management in Team Services and TFS](https://www.visualstudio.com/en-us/docs/package/overview)