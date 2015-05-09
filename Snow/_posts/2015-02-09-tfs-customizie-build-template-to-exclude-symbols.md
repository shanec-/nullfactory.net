---
layout: post
title: Customize a Build Template to Exclude Symbols From Being Published to the Symbol Server
category: Team Foundation Server, Team Build
---

This post was inspired by a feedback provided by one of the commenters on [this post](http://www.edsquared.com/2011/02/12/Source+Server+And+Symbol+Server+Support+In+TFS+2010.aspx). The requirement was to exclude certain third party symbols from being published to the symbol server. I thought I would take upon the challenge to implement this.

As suggested in the same comment thread, my approach would revolve around explicitly setting the `FileList` property in the `PublishSymbols` activity. I would set it to a list that includes only the symbols that I want published. And the symbols I want to be excluded would be handled through a wild card filter that is passed as a build template parameter.

## Pre-Requisites

- A properly configured symbol server. Check out my previous on [automatically publishing debug symbols](/2015/01/setting-up-tfs-source-symbol-servers/).

## Customization Steps

My template is based on the default TFS template `TfvcTemplate.12.xaml` and customized with the following changes:

<!--excerpt-->

1. Start off by adding the build template parameter that accepts filter:
	- `ExclusionFilter` - *Text that would be used as part of the wild card filter.*

		![Build Process Parameter](/images/posts/ExcludeSymbols/10_Parameters.png)

1. Next, the variables that would be needed:
	- `CustomAllSymbolsList` - *Temporarily store the symbol files that are found*
	- `CustomExcludedSymbolsList` - *Temporarily store the symbols that need to be excluded.*
	- `CustomFilteredSymbolsList` - *The final list of symbols that would be published to the symbol server.*
	- `CustomBinDirectory` - *Temporary store the Binaries Directory path.*

		![Build Process Variables](/images/posts/ExcludeSymbols/20_Variables.png)
 
1. Now get the `WellKnowEnvironmentalVariables.BinariesDirectory` value using the `GetEnvironmentalVariable<T>` activity.
	
	![GetEnvironmentalVariable<T>](/images/posts/ExcludeSymbols/40_EnvronmentalVariableActivity.png)

	![Built-In Environmental Variable](/images/posts/ExcludeSymbols/30_EnvironmentalVariable.png)

1. Get a list of all the available symbols from the build folder.
	- This is done using a `FindMatchingFiles` activity with a `String.Format("{0}\**\*.pdb", CustomBinDirectory)` pattern.

	![Find All Symbols](/images/posts/ExcludeSymbols/45_FindAllSymbols.png)
 
1. Get a list of of symbols that need to be excluded from the Symbol Server.
	- Again, a `FindMatchingFiles` activity with `String.Format("{0}\**\*{1}*.pdb", CustomBinDirectory, ExclusionFilter)` pattern.

	![Find Symbols to be excluded](/images/posts/ExcludeSymbols/47_FindSymbolsToExclude.png)

1. Since the `FindMatchingFiles` activity returns an `IEnumberable<string>` we create a new list that we can actually manipulate.
	
	![Find Symbols to be excluded](/images/posts/ExcludeSymbols/50_InitializeSymbolList.png)

1. Add all the symbols to the new list.

	![Add symbols](/images/posts/ExcludeSymbols/60_AddToCollection.png)

1. Remove the symbols that need to be excluded. 
	
	![Remove symbols](/images/posts/ExcludeSymbols/70_RemoveFromCollection.png)

1. Pass the filtered collection into the `PublishSymbols` activity via the `FileList` property.

	![PublishSymbols Activity](/images/posts/ExcludeSymbols/80_PublishSymbols.png)

	![PublishSymbols, FileList](/images/posts/ExcludeSymbols/90_FilteredPublishSymbols.png)

## Using the Build Definition

Create a new build definition using the new template and provide valid values for `Path to publish symbols`and `ExclusionFilter` parameters. 

![Edit build definition](/images/posts/ExcludeSymbols/100_EditBuildDefinition.png)

Finally, queue a new build and verify that the symbols are excluded the symbols store. 

The final customized template can be [downloaded here](https://github.com/shanec-/Nullfactory-TfsBuildExtensions/blob/master/src/Template/ExcludedSymbolsTfvcTemplate.12.xaml).

## References

- [MSDN - Index and publish symbol data](http://msdn.microsoft.com/en-us/library/hh190722.aspx)
- [MSDN - Embed version control paths and versions into the symbol data in Your PDB files (IndexSources activity)](http://msdn.microsoft.com/en-us/library/gg265783.aspx#Activity_IndexSources)
- [MSDN - Publish symbols to a SymStore symbol store (PublishSymbols activity)](http://msdn.microsoft.com/en-us/library/gg265783.aspx#Activity_PublishSymbols)
- [Ewald Hofman | Customize Team Build 2010 – Part 2: Add arguments and variables](http://www.ewaldhofman.nl/post/2010/04/27/Customize-Team-Build-2010-e28093-Part-2-Add-arguments-and-variables.aspx)
- [MSDN- Customize your build process template](http://msdn.microsoft.com/en-us/library/dd647551.aspx)
- [MSDN - Team Foundation Build environment variables](http://msdn.microsoft.com/en-gb/library/hh850448.aspx)
- [MSDN - GetEnvironmentVariable<T> Class](https://msdn.microsoft.com/en-us/library/dn231755.aspx)
- [MSDN - WellKnownEnvironmentVariables.BinariesDirectory Field](https://msdn.microsoft.com/en-us/library/microsoft.teamfoundation.build.activities.extensions.wellknownenvironmentvariables.binariesdirectory.aspx)