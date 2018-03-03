---
layout: post
title: YAML Builds and Branching in Visual Studio Team Services
category: Visual Studio Team Services, ALM, Git
---

In my previous post I explored the the concept of treating build as a first class code citizen and how to automatically setup up an YAML based continuous integration (CI) build in Visual Studio Team Services (VSTS). 

At the end of that exercise I was curious as to how VSTS handles build definitions when you throw branching into the mix. I could not find much information on the topic, so I thought I might try it out myself.

## Branches and CI Builds

1. Prepare the project structure place the you build definition `.vsts-ci.yml` in the root of the repository. Read [my previous post](/2018/01/build-pipeline-as-code-yaml-based-ci-build-for-dynamics-365-solutions/) for setting up an exported CI build.

2. Add an echo step so we can uniquely identify this build branch.

    ![Triggered Echo](/images/posts/YamlBuildBranch/5_TriggeredEchoMaster.png)
<!--excerpt-->
3. Commit changes to the `master` branch and push it to the remote repository.

4. Verify that VSTS picked up the build and provisioned the CI build as expected. And that it triggered an initial build.

    ![Build Provisioned](/images/posts/YamlBuildBranch/10_BuildProvisioned.png)

5. Verify that our echo statement worked.

    ![Build Echo Statement](/images/posts/YamlBuildBranch/15_BuildEchoStatement.png)

6. Now let's update the build trigger to include all feature branches we create. Edit the build definition and navigate to the triggers tab add the following branch filter: `features/*`. This means that the build would trigger on each of the `feature/` topic branches.

    ![Edit Build Definition](/images/posts/YamlBuildBranch/20_EditBuildDefinition.png)

7. Navigate to the triggers tab and add a new branch filter.

    ![Add Branch Filter](/images/posts/YamlBuildBranch/30_AddBranchFilter.png)

8. Save the changes.

9. Next, create a new topic branch and switch to it. I am going to call it `features/complex-feature1`.

   ​	`git checkout -b 'features/complex-feature1'`

10. Push the new branch to the remote server.

  ​	`git push -u origin 'features/complex-feature1' `

11. Verify that the build definition got triggered against the new branch.

    ![Branch triggered on New Branch](/images/posts/YamlBuildBranch/40_BranchBuildTriggered.png)

12. Now let's make a change to the `.vsts-ci.yml`. In order to keep things simple, I am just going to update the echo statement with the branch name.

    ![Branch triggered on New Branch](/images/posts/YamlBuildBranch/50_UpdatedEcho.png)

13. Once again, let's commit and push the changes to the remote server.

14. Verify that a new build was triggered.

    ![Updated Build Trigger](/images/posts/YamlBuildBranch/60_UpdatedBuildTriggered.png)

15. Confirm that the branch specific version of the build was executed.

    ![Branch triggered on New Branch](/images/posts/YamlBuildBranch/70_BranchBuild.png)


This confirms that VSTS honors and executes the appropriate build definitions on the appropriate branch. Super cool!

## Final Thoughts

The ability to branch build definitions gives team members the ability to customize and test build changes in isolation. It also minimizes the disruption to on going development until the changes are ready to be merged and shared with the rest of the team.

### Path Filters

It doesn't seem to be possible to setup path filters from the YAML file. That being said, it can still be setup by editing the build definition after the provisioning. 

This can be done by navigate to the `Triggers` tab in the build definition and adding the path filter.

![Path Filters](/images/posts/YamlBuildBranch/80_PathFilter.png)

In the above example, I want to make sure that the CI build does not trigger when I make changes to the support tools. 

### Scheduling Builds

Schedule builds also needs to be setup in a similar fashion to path filters. 

![Build Schedule](/images/posts/YamlBuildBranch/90_BuildSchedule.png)

### Multiple Build Definitions and Manual Setup

What happens when you have multiple `yml` files within a single repository? I hadn't even considered this until I stumble across it in the documentation. 

Consider an example where the project already contains your wired up CI build but you'd like to use the second dedicated build for your production releases. 

This is certainly possible - if VSTS finds a a file named `.vsts-ci.yml` in the root path then it attempts to wire it up as a CI [build automatically](https://docs.microsoft.com/en-us/vsts/build-release/actions/build-yaml#automatically-create-a-yaml-build-definition), but the additional builds needs to be [setup manually](https://docs.microsoft.com/en-us/vsts/build-release/actions/build-yaml#manually-create-a-yaml-build-definition). 

1. Start by creating a new build definition from within VSTS and choose `YAML` as the template. Confirm the selection by clicking the `Apply` button.

   ![Build Schedule](/images/posts/YamlBuildBranch/100_NewBuildTemplate.png)

2. Next, click the ellipsis button `...` and select the path to the YAML file and click `OK` to confirm.

    ![Build Schedule](/images/posts/YamlBuildBranch/110_ProdBuild.png)

3. Once saved, you can trigger the build as you would any other.

I can also confirm that VSTS treats YAML builds as regular builds and they can be used as the source for your release definitions.

### Change Tracking

So the changes YAML file itself is tracked by `git`, but does VSTS track the changes we do to the build definition after provisioning? Yes, it does. Under the covers, it looks like VSTS sets up an json-exportable definition of the build. This is not a replication of your `YAML` build tasks, but rather maintains everything thats **not** configurable in the `YAML` file.

A history of all changes done to the definition can be viewed and compared from the history tab.

![Build History](/images/posts/YamlBuildBranch/120_BuildHistory.png)

I, for one, have become a fan of YAML builds and can't wait for this feature to come out of preview and become a permanent.

## References

- [Define a CI build process for your Git repo | Microsoft Docs](https://docs.microsoft.com/en-us/vsts/build-release/actions/ci-build-git)
- [CI Build in code using YAML | Microsoft Docs](https://docs.microsoft.com/en-us/vsts/build-release/actions/build-yaml)
- [Build variables | Microsoft Docs](https://docs.microsoft.com/en-gb/vsts/build-release/concepts/definitions/build/variables?tabs=batch)
- [Build definition triggers | Microsoft Docs](https://docs.microsoft.com/en-us/vsts/build-release/concepts/definitions/build/triggers)
- [CI Build in code using YAML - Automatic Build Definition| Microsoft Docs](https://docs.microsoft.com/en-us/vsts/build-release/actions/build-yaml#automatically-create-a-yaml-build-definition)
- [CI Build in code using YAML - Manual Build Definition| Microsoft Docs](https://docs.microsoft.com/en-us/vsts/build-release/actions/build-yaml#manually-create-a-yaml-build-definition)

