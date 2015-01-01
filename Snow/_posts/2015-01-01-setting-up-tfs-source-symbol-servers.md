---
layout: post
title: Setting up Source and Symbol Servers in Team Foundation Server
category: Team Foundation Server
---

## What are debug symbols?
Debug symbols are artifacts that a debugger can use in order to better debug an application. Within the.NET ecosystem these are managed through PDB files. The PDB files contain information about the source file name, line numbers as well as local variable names.

As a software solution evolves, it is likely that multiple versions of it gets deployed into different production systems. And once the software is out in the wild, it becomes important that the developers can react to issues discovered by debugging specific versions. In order to do this effectively, it is important that the debug symbols themselves be treated as an first class artifact of the build and that it is readily accessible. Team Foundation Server (TFS) achieves this via a Source Server and Symbol Server.

## The Source Server and Symbol Server
A Source Server component is essentially letting TFS know that we would be retrieving specific versions of source files and that it should be indexed. And a Symbol Server is a fancy name for a networked file share location containing the multiple versions of symbols [[read more]](http://blogs.msdn.com/b/jimlamb/archive/2009/06/15/symbol-and-source-server-in-tfs-2010.aspx).

A TFS build definition will be configured in order to automatically index sources and publish symbols [[read more]](http://blogs.msdn.com/b/adamroot/archive/2009/06/17/source-server-and-symbol-server-features-in-team-foundation-server-2010-beta-1.aspx).

<!--excerpt-->

## Configuration
There are two components that need to be configured for an effective workflow. The build definition; this would be instructing TFS to index the sources and publish the symbols to a share location. Then instruct each of the clients (Visual Studio in this case) to use the symbol cache.

### Build Definition (Server)
Configuring the build definition is pretty straight forward:

1. Setup a file share on a server other than the build agent. This server should be able to be accessed from the build agent as well as any clients that would be debugging the code.
1. Grant build service account write access permission to the folder.  
1. Grant all project contributors read access to this folder.
1. Edit the build definition by providing a value to the `Publish Symbols > Path to publish symbols` property. Set it to the previously created network folder.

![Setup the Build Definition](/images/posts/SetupSymbolServer/1_PathToPublishSymbols.png)

### Clients
Now that the server is setup, we configure visual studio to take advantage of the symbol store:

1. Open up Visual Studio and navigate to `Tools > Options > Debugging`.
1. Under the `General` section, check the `Enable source server support` flag.
![Enable Source Server Support](/images/posts/SetupSymbolServer/2_VsEnableSourceServerSupport.png)
1. Next, navigate to `Symbols` node.
1. Create a new symbol file location entry and set the cache directory to the previously created share location.
![Setup the Symbol Cache](/images/posts/SetupSymbolServer/3_VsSetupSymbolCache.png)

## Conclusion

The debug symbols would automatically be published to the share location every time a new build is triggered. The Visual Studio clients should be able to use the appropriate symbols when debugging.

## References
* [Source Server and Symbol Server Support in TFS 2010 - Ed Squared](http://www.edsquared.com/2011/02/12/Source+Server+And+Symbol+Server+Support+In+TFS+2010.aspx)
* [MSDN Blogs - Source Server and Symbol Server Features in Team Foundation Server 2010 - Anything's Possible](http://blogs.msdn.com/b/adamroot/archive/2009/06/17/source-server-and-symbol-server-features-in-team-foundation-server-2010-beta-1.aspx)
* [MSDN Blogs - Enabling Symbol and Source Server Support in TFS Build 2010](http://blogs.msdn.com/b/jimlamb/archive/2009/06/15/symbol-and-source-server-in-tfs-2010.aspx)
* [MSDN Blogs - Making debugging easier: Source Indexing and Symbol Server](http://blogs.msdn.com/b/buckh/archive/2011/04/11/making-debugging-easier-source-indexing-and-symbol-server.aspx)
