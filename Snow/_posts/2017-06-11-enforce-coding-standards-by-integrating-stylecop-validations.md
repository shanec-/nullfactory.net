---
layout: post
title: Enforce Code Standards by Integrating StyleCop Validations
category: ALM, StyleCop
---
StyleCop is a configurable analysis library that ensures code produced conforms to it's coding standard. While the benefits of a consistent coding standard is clear, each team has there own strong opinions how it should be enforced. 

In my previous projects I've used the `StyleCop.MsBuild` package and have been eager to try out the code analyzer version of the tool. In this post, I will start off with a quick run down of my attempt at integrating it into a solution.

## Prerequisites
 
- Visual Studio Professional 2015 or higher. Its not really necessary, but makes for an easier setup and nicer integration with the Roslyn base analyzer. 
- If integrating with an SCM or Automated build -  Visual Studio Team Services (VSTS) using git. The steps are not limited to VSTS, but its just my personal preference.

## Installation

Installation itself is pretty simple - just install the `StyleCop.Analyzers` nuget package by running the following command in the package manager console witihin Visual Studio:

    Install-Package StyleCop.Analyzers

That's it. That all we need to do to get started.

## Warning Vs. Error

By default StyleCop treats suggestions as warnings, but I prefer to go strict mode and treat all style cop validations as errors. This means that build actually fails if the code does not adhere to the style validations. My justification is that its always cheaper to fix these issues early in the development cycle as well as minimizing ["broken windows syndrome"](https://blog.codinghorror.com/the-broken-window-theory/).

When using the older `StyleCop.MSBuild` package, I could just set the `StyleCopTreatErrorsAsWarnings` environmental variable in the development machine or the team build and it will ensure that the the scope is limited to the StyleCop errors only. Unfortunately, with the Analyzers project it does not seem to be that simple. There exists an `TreatWarningsAsErrors` environmental variable, but this is global and treats any warnings as errors - not what I want.

<!--excerpt-->

So my plan is to create two rule sets and tie them down to two individual configurations. I will create one rule set used in the debug configuration and then a stricter one for the release configuration.

Now let's configure the rules:

1. Let's create a very simple console application called `AnalyzersDemo`and open it up in Visual Studio.
1. Install the `StyleCop.Analyzers` package into the project.
1. Next, navigate to the `Analyze > Configure Code Analysis > For AnalyzersDemo]` menu option.

    ![Code Analysis](/images/posts/StyleCopInteg/10_configureanalysis.png)

1. In the resulting dialog, accept the default values by clicking the `Open` button. This will open a dialog that allows us to configure the behaviour of the rule sets.

    ![Code Analysis Customize](/images/posts/StyleCopInteg/20_opencustomize.png)

1. Select all the warnings related to `StyleCop.Analyzers` and update the Action to `Error`. Notice that a new rule set file has been created.

    ![Code Analysis Customize Rules](/images/posts/StyleCopInteg/30_customizerules.png)

1. Duplicate the rule set file and rename them `BaseRuleset.ruleset` to `StrictRuleset.ruleset`. 
1. Move them them up as solution items. This is so that we can share the same rule sets against all the projects.
1. Edit the files and give the name and descriptions more meaningful values.
1. Remove all nodes except for the `<RuleSet>` root node from the `BaseRuleset` file so that it will revert the default action level.
1. Next, associate the two rule sets with the different configurations; the `BaseRuleset` and `StrictRuleset` to the `debug` and `release` configuration respectively.

    ![Debug RuleSet](/images/posts/StyleCopInteg/40_debugrules.png)

    ![Release RuleSet](/images/posts/StyleCopInteg/50_releaserules.png)

1. This is the final project structure:

    ![Project Structure](/images/posts/StyleCopInteg/60_projectstructure.png)

Now the strict validation is performed only when a `release` build is performed.

## Final Thoughts

One may argue that the "draconian mode" of enforcing styles is not worth the potential disruption it causes.  So let's try to compromise - If your source control strategy uses multiple branches (which you should always do, by the way), the strict mode can be enforced as a gated check-in. This means that the rules are enforced only when a pull-request is made into the `master` branch. Refer my [previous post](/2017/05/gated-checkins-in-vsts-using-tfsvc-and-git/) on setting up a gated branch policy.

More information on customization of the StyleCop rules can be found [here](https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/documentation/Configuration.md). 

And finally, here is a build manually executed with an explicit configuration:

![Queue Build](/images/posts/StyleCopInteg/65_queuebuild.png)

The failure in a `release` configuration:

![Release Build](/images/posts/StyleCopInteg/70_releasebuild.png)

And warnings in `debug`:

![Debug Build](/images/posts/StyleCopInteg/80_debugbuild.png)

## References

- [StyleCop - Documentation](https://stylecop.codeplex.com/wikipage?title=Setting%20Up%20StyleCop%20MSBuild%20Integration)
- [Using, configuring and distributing Roslyn analysers in teams](http://tech.marketinvoice.com/2015/09/06/using-roslyn-analysers/)
- [NuGet Gallery | StyleCop.MSBuild 4.7.55](https://www.nuget.org/packages/StyleCop.MSBuild/)
- [GitHub - DotNetAnalyzers/StyleCopAnalyzers: An implementation of StyleCop rules using the .NET Compiler Platform](https://github.com/DotNetAnalyzers/StyleCopAnalyzers)
- [The Broken Window Theory](https://blog.codinghorror.com/the-broken-window-theory/)