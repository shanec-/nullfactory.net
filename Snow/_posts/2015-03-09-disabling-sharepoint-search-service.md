---
layout: post
title: Disabling the SharePoint Search Service
category: SharePoint, Windows
---

I've The search service always tends always be going
Working with SharePoint 2013 and I've noticed the search service going out of control in my development box.

[This](http://www.falconitservices.com/support/KB/Lists/Posts/Post.aspx?ID=182) seems to be most effective way to completely disable the SharePoint 2013 search service. 

While changing the password to an invalid one works great, it is important to update the user account from a domain to a local user account. If not, the constant invalid password requests could trigger your account lock out threshold policy in the domain.

So for convenience sake I created two batch files - one to disable the service and another to bring it back up.  

Here's the script to disable the search service:

	net user searchDummy Sup3rSecr3tPwd /add /passwordchg:no /expires:never
	net stop SPSearchHostController
	net stop OSearch15
	sc config "SPSearchHostController" obj= ".\searchDummy" password= "WrongPwd"
	sc config "OSearch15" obj= ".\searchDummy" password= "WrongPwd"
	taskkill /im:noderunner.exe /f

Notice that: 
	- The first command creates the dummy user in the local machine - this will substitute the domain service account.
	- The password is intentionally set to an incorrect one when assigned to the service.
	- The last command is to force terminate any instances of noderunner and release some memory.

And here are the commands I use to start the service up again:

	sc config "SPSearchHostController" obj= "domain\searchservice" password= "CorrectPwd"
	sc config "OSearch15" obj= "domain\searchservice" password= "CorrectPwd"
	net start OSearch15
	net start SPSearchHostController

## References
- http://www.falconitservices.com/support/KB/Lists/Posts/Post.aspx?ID=182
- https://technet.microsoft.com/en-us/library/hh994574.aspx
- https://technet.microsoft.com/en-us/library/cc990290.aspx
- http://stackoverflow.com/questions/308298/how-to-set-windows-service-username-and-password-through-commandline
- http://blogs.msdn.com/b/ssehgal/archive/2009/06/01/configuring-windows-services-using-command-prompt.aspx
- http://www.windows-commandline.com/add-user-from-command-line/
- http://www.windows-commandline.com/cmd-net-user-command/