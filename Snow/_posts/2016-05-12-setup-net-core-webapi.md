---
layout: post
title: Setting up .NET Core WebApi to work
category: .net core, WebApi
published: draft
---

I've been experimenting with SPA applications recently and have been wanting to create the back end service using WebApi with the latest .NET core using VS Code

## Pre-Requisites

First off, let's get the pre-requisites tools and frameworks installed.

1. Install NodeJS - [https://nodejs.org/en/download/](https://nodejs.org/en/download/).

1. Install the latest version of the .NET core - [https://www.microsoft.com/net/core#windows](https://www.microsoft.com/net/core#windows).

1. Download and install VSCode - [https://code.visualstudio.com](https://code.visualstudio.com).

1. Install the yeoman generator via the node package manager.

		npm install -g yo

1. Install the asp.net generator for yeoman.

		npm install -g generator-aspnet

1. (Optional) Install the `C# for Visual Studio Code` extension for VSCode.

	I found that I had somehow "broken" the C# extension, so I had to re-install it in order to get it working - YMMV. I did this by running `ext install csharp` in the VSCode quick open prompt (`Ctrl+P`).

When I first attempted to debug the solution using VSCode I got the following warning:

    WARNING: Could not load symbols for 'NfSample.dll'. 'D:\temp\NfSample\bin\Debug\netcoreapp1.0\nfSample.pdb' is a Windows PDB. These are not supported by the cross-platform .NET Core debugger.

This appears to be due to an incompatibility with the latest version of .net core and vscode on Windows. Read more about it [here](https://github.com/OmniSharp/omnisharp-vscode/issues/82). Luckily, [the fix itself is simple](https://github.com/OmniSharp/omnisharp-vscode/wiki/Portable-PDBs#net-cli-projects-projectjson); enable the following build option in the `project.json`:

	"buildOptions": {
	    "debugType": "portable"
	},

## References
- [omnisharp-vscode/debugger.md at master · OmniSharp/omnisharp-vscode](https://github.com/OmniSharp/omnisharp-vscode/issues/82)
- [Portable PDBs · OmniSharp/omnisharp-vscode Wiki](https://github.com/OmniSharp/omnisharp-vscode/wiki/Portable-PDBs#net-cli-projects-projectjson)
- [C# | Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)