---
layout: post
title: Disabling the SharePoint Search Service
category: SharePoint, Windows
---

If left unchecked, the SharePoint search service to attempts to consume most of the available memory in a resource constrained development box. I usually disable it when I am not actively working with it.

[This seems to be most effective](http://www.falconitservices.com/support/KB/Lists/Posts/Post.aspx?ID=182) way to completely disable the service. Although changing the password to an invalid one works as expected, it is important to replace the user account from a domain to a local user account as well. If not, the constant invalid login attempts could trigger your account lock out threshold policy in the domain.

So for convenience sake I created two batch files - one to disable the service and another to bring it back up.  

<!--excerpt-->

## Disable the Search Service

The commands used to disable the search service:

	net user searchDummy Sup3rSecr3tPwd /add /passwordchg:no /expires:never
	net stop SPSearchHostController
	net stop OSearch15
	sc config "SPSearchHostController" obj= ".\searchDummy" password= "WrongPwd"
	sc config "OSearch15" obj= ".\searchDummy" password= "WrongPwd"
	taskkill /im:noderunner.exe /f

Notice that: 

- The first command creates the dummy user in the local machine - this will substitute the domain service account.
- The password is intentionally set to an incorrect one when assigned to the service.
- The last command is to force terminate any instances of `noderunner.exe` and release some memory.

## Enable the Search Service

The commands used to start the service up again:

	sc config "SPSearchHostController" obj= "domain\searchservice" password= "CorrectPwd"
	sc config "OSearch15" obj= "domain\searchservice" password= "CorrectPwd"
	net start OSearch15
	net start SPSearchHostController

## References
- [How to Disable SharePoint 2013 Search - IT Knowledgebase](http://www.falconitservices.com/support/KB/Lists/Posts/Post.aspx?ID=182)
- [Technet -Account lockout threshold](https://technet.microsoft.com/en-us/library/hh994574.aspx)
- [Technet - Sc Config](https://technet.microsoft.com/en-us/library/cc990290.aspx)
- [StackOverflow - how to set windows service username and password through commandline](http://stackoverflow.com/questions/308298/how-to-set-windows-service-username-and-password-through-commandline)
- [MSDN Blogs - Configuring Windows services using Command Prompt - Sidharth Sehgal's blog](http://blogs.msdn.com/b/ssehgal/archive/2009/06/01/configuring-windows-services-using-command-prompt.aspx)
- [Add new user account from command line (CMD)](http://www.windows-commandline.com/add-user-from-command-line/)