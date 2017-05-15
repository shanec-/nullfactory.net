---
layout: post
title: Gated Check-ins in Visual Studio Team Services using TFSVC and Git
category: Visual Studio Team Services, TFSVC, ALM
---

A continuous integration build ensures that the code within a source control repository can be compiled successfully after a commit. [It is defined as](https://www.thoughtworks.com/continuous-integration):

> Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early. 

Where as CI builds occur after a commit has been merged into the repository, a gated check-in is a "pessimistic" process that would treat a commit as completed **only** if the result of it being merged with the repository is a successful build.

In this post I will try my hand in setting up a gated check-in using Visual Studio Team Services (VSTS). I will start off with TFSVC as the underlying version control system and then replicate the same functionality using Git.

## Configuring a Gated Check-In Using TFSVC  

*Prerequisite: A repository already exists in the remote server with project code checked-in. I will be using one of my demo CRM projects.*

1. Define the steps for the build that would act as the verification build.

	![Verification Build](/images/posts/TfsvcGatedCheckIns/10_CIBuild.png)

1. Navigate to the `Triggers` tab within the build definition and enabling the `Gated Check-In` trigger.

	![Gated Check-In](/images/posts/TfsvcGatedCheckIns/20_GatedCheckIn.png)
	
<!--excerpt-->

1. Leave the `Run continuous integration triggers for committed changes` unchecked. If a CI build has been configured and [this option was checked](https://www.visualstudio.com/en-us/docs/build/define/triggers) then an additional CI build would run after a successful gated check-in build.

### Testing Out the Process

1. Make a change to the code that causes an compilation error. 
1. Check-in this code into the version control. Upon clicking the `Check In` button within Visual Studio, the following warning dialog appears indicating that the new check-in would be stored as a shelveset until its successful completion: 

	![Gated Check-In Warning](/images/posts/TfsvcGatedCheckIns/30_GatedWarning.png)
	
1. Uncheck the `Preserve my pending changes locally` option and click on the `Build Changes` button to kick off the build.
1. Next, navigate to builds definition folder and confirm that the verification build was in fact triggered and that it failed as expected.

	![Failed Gated Build](/images/posts/TfsvcGatedCheckIns/40_GatedCIBuild.png)
	
1. Since we did not opt-in to preserve our check-ins locally, let's retrieve it from the generated shelveset. Back in Visual Studio, navigate to the`Team Explorer` window and from the `Actions` menu click the `Find Shelvesets` 

	![Failed Gated Build](/images/posts/TfsvcGatedCheckIns/50_FindShelveset.png)

1. Select the last shelveset that failed the build - it would be prefixed with `Gated_`.
	
	![Shelvesets Assigned](/images/posts/TfsvcGatedCheckIns/60_ShelvesetAssigned.png)
	
1. Fix the compilation errors we introduced previously and check in the changes.

	![Check-in Corrections](/images/posts/TfsvcGatedCheckIns/70_Recheckin.png)
	
1. Verify that the new check in and resultant gated-build is successful. 

	![Successful Gated Build](/images/posts/TfsvcGatedCheckIns/80_BuildSuccess.png)
	
1. Now that the check-in has been merged, get the latest version of the code using Visual Studio.

## Configuring a Gated Check-In Using Git

Branch Policies are used to implement gated check-ins in git. It is a feature in VSTS that provides rules to govern the activity within a branch and ensure the code quality and change management standards are followed. 

There are many [policies available](https://www.visualstudio.com/en-us/docs/git/branch-policies) but I am interested only in the ones related to when a pull request is made.

Given the way that git operates, it cannot be configured to have a gated check-in against every commit as we did with TFSVC. Instead the policy would be enforced during the pull requests between the topic branch to the `master` branch.

The following steps describe the process of setting up a gated check-in on the `master` branch:

*Prerequisite: A repository already exists in the remote server with project code checked-in. I will be using one of my demo CRM projects.*

1. Start off by defining a build that would act as the verification.

	![Verification Build](/images/posts/GitGatedCheckIns/05_verificationbuild.png)

1. Navigate to the repository.
1. Click on the `Branches` menu.
1. Click on the ellipsis against the `master` branch and on the menu and select `Branch Policies`.

	![Update Branch](/images/posts/GitGatedCheckIns/10_updatebranch.png)

1. Under the `Automatically build pull requests` menu check the `When team members create or update a pull request into the master branch, queue this build: ` option.
1. Then select the build that you want to use to perform the verification.

	![Branch Policies](/images/posts/GitGatedCheckIns/20_setupbranchpolicy.png)

1. Click `Save Changes` button to confirm the changes.

### Testing out the Policy

Now let's test our branch policy:

1. On the development machine, create a new local branch called `development`.
1. Commit a change that causes a compilation error. 
1. Push the branch and changes to the remote repository.

	![Development Branch](/images/posts/GitGatedCheckIns/30_devbranch.png)

1. Next, log into VSTS and navigate to the `Branches` menu.
1. Click on the ellipsis against the `development` branch to open up the context menu. 
1. Select the `New pull request` option.

	![New pull request](/images/posts/GitGatedCheckIns/40_newpullrequest.png)

1. Provide a meaningful `Title` and `Description` and click on the `Create` button.

	![New pull request](/images/posts/GitGatedCheckIns/50_pullrequestdialog.png)

1. Confirm that a new verification build has been queued for the pull request.

	![Verification Build](/images/posts/GitGatedCheckIns/60_verifybuild.png)

1. Confirm that this build failed.

	![Confirm Build Failure](/images/posts/GitGatedCheckIns/70_confirmbuildfailed.png)

1. Now let's fix the code, commit and push the change in to the repository.

	![Fix the build](/images/posts/GitGatedCheckIns/80_fixbuild.png)

1. Verify that the build policy has detected the new commit and automatically associated it with the pull request. 
1. Confirm that a new build was triggered as a result and that its successful. The successful build completes the pull request.

	![Complete Pull Request](/images/posts/GitGatedCheckIns/90_completedpullrequest.png)

## References

- [Continuous integration | ThoughtWorks](https://www.thoughtworks.com/continuous-integration)
- [Build definition triggers](https://www.visualstudio.com/en-us/docs/build/define/triggers)
- [Check in to a folder that is controlled by a gated check-in build process](https://www.visualstudio.com/en-us/docs/tfvc/check-folder-controlled-by-gated-check-build-process)
- [Git Branches and Policies in Team Foundation Server 2015 | Visual Studio 2015 Final Release Event | Channel 9](https://channel9.msdn.com/Events/Visual-Studio/Visual-Studio-2015-Final-Release-Event/Git-Branches-and-Policies-in-Team-Foundation-Server-2015)
- [Gated checkin for Git: Using Branch Policies to run a build in VSTS and TFS – Buck Hodges](https://blogs.msdn.microsoft.com/buckh/2016/03/20/gated-checkin-for-git-using-branch-policies-to-run-a-build-in-vsts-and-tfs/)
- [Protect your Git branches with policies | Team Services & TFS](https://www.visualstudio.com/en-us/docs/git/branch-policies)