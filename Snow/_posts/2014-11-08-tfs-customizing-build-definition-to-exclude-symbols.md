---
layout: post
title: Team Foundation Server - Customizing a Build Definitions to Exclude Symbols in Symbol Server
category: ALM, Team Foundation Server
published: draft
---

## Overview

In my previous post I wrote about updating the build definition in order to automatically publish the symbols. 

This post was inspired by a feedback provided by one of the commenter on [this post](http://www.edsquared.com/2011/02/12/Source+Server+And+Symbol+Server+Support+In+TFS+2010.aspx). The individual required that certain third party symbols be omitted from being published to the Symbols Server and pushed to clients.
I thought I would take upon the challenge to implement the same.

## Pre-Requisites

This assumes that you have already successfully setup a symbol server and  


I have done successfully setup Symbol server in TFS. But i need to exclude third part pdb file at that time of publish to Symbol server.
The options available in client side, but i need to setup in server side (TFS build).

Hi Rajesh,

I don't of a particularly easy way to handle this scenario off the top of my head. It would involve customizing the build process template and removing the specific .PDB entries that you don't want to be published from the FileList collection that is passed into the Publish Symbols build workflow activity.


http://msdn.microsoft.com/en-us/library/gg265783.aspx#Activity_PublishSymbols



## References

Why publish symbols