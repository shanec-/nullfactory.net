---
layout: post
title: YAML Build Support for generator-nullfactory-xrm
category: generator-nullfactory-xrm, Visual Studio Team Services, ALM, Dynamics CRM, Dynamics CRM Online, Yeoman, npm, nodejs
---

If you've read my previous posts on the topic, you might have already guessed that I am a big fan of maintaining builds as code artefacts. I was super excited when the Visual Studio Team Services (VSTS) team announced YAML support and I knew right then that this had to be part of `generator-nullfactory-xrm`. 

With version `1.6.0`, I've added the ability to generate tailored YAML build definitions into your Dynamics 365 projects. What this means is that you get rich CI build support right out of the gate! No more excuses for not having a CI build in your project!

![CI Builds for Everyone](/images/posts/YamlBuildGeneratorSupport/hero.jpg)

The option to generate a CI build is defined as a sub-generator. This is intentional as I didn't want to automatically assume that everyone would be using VSTS as their source control.

Here's the quick rundown of the steps necessary to get things up and running. What we're aiming for is a layered approach:

1. First generate the default project structure.
2. Next, generate the YML file using a sub-generator.
3. Push changes to remote VSTS `git` repository.
4. Verify that VSTS provisioned the YML file as the CI build.
4. ... 
5. Profit!

<!--excerpt-->

## Prerequisites

1. Opt-in to the preview build feature in VSTS. [Read my previous post](/2018/01/build-pipeline-as-code-yaml-based-ci-build-for-dynamics-365-solutions/) on how to do this.

    *Fair warning that as of writing this post, the YAML builds feature within VSTS is still in preview. While very unlikely, there's always the possibility that things might break due to future change.*

2. A `git` based repository in VSTS - YAML builds only work with git repositories.

## The Setup

Assumption that nullfactory-xrm generator and [Yeoman](http://yeoman.io/) itself has alredy been installed on the client computer being worked on and that a Dynamics CRM solution is ready to be synced.

1. Setup the project using `generator-nullfactory-xrm` using the default generator. [Read this post](/2016/10/release-strategy-for-dynamics-crm-prepping-part-1/) for more detail steps.

        yo nullfactory-xrm

	![Default Generator](/images/posts/YamlBuildGeneratorSupport/10_defaultgenerator.png)

2. Next, run the generator once more from the same folder location, and this time let's use the `cibuild` sub-generator:

        yo nullfactory-xrm:cibuild

	Provide the same solution name you provided in step 1. And choose `Visual Studio Team Services` as the type of source control being used.

	![Default Generator](/images/posts/YamlBuildGeneratorSupport/20_generatecibuild.png)

This would have generated a file called `.vsts-ci.yml`. And that's it - once your changes are checked in and pushed to VSTS, the YML file would be automatically provisioned as the default CI build.