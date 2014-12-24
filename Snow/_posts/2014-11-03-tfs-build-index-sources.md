---
layout: post
title: Team Foundation Server - Team Build Index Sources
category: ALM, Team Foundation Server
---

###What are debug symbols?
Debug symbols are artifacts that a debugger can use in order to better debug an application.

Within the.NET ecosystem these are managed through PDB files. The PDB files contain information about the source file name, line numbers as well as local variable names. 


###Symbol Server
As a software solution evolves, it is likely that multiple versions of it get deployed into different production systems. And once the software is out in the wild, it becomes important that the developers can react to issues discovered by debugging specific versions of the software. In order to do this effectively, it is important that the debug symbols themselves be treated as an important artifact of the build. 

Source Server aids in this by enabling the the debug symbols (PDB) of multiple versions to be stored in a central location. A TFS build definition can be configured in order to automatically publish the symbols.

###Configuration
The following steps are necessary in order to configure the build definition. 

####Server
1. Setup a file share on a server other than the build agent. This server should be able to be accessed from the build agent as well as any clients that would be debugging the code.
2. 

####Clients

1. Open up Visual Studio and go into options
2. 

###References
http://www.edsquared.com/2011/02/12/Source+Server+And+Symbol+Server+Support+In+TFS+2010.aspx