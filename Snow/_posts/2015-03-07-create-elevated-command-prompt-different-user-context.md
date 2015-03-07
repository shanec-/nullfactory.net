---
layout: post
title: Create an Elevated Command Prompt Running Under a Different User Context
category: Security
---

User Account Control (UAC) is a security mechanism that is available in most modern versions of Windows. It restricts the ability to make changes to a computer environment without the explicit consent of an administrator. And as is with most production environments, it is very likely that this feature is already enabled. And just as likely, is the need to execute applications and commands under a different user within an elevated context during application deployment or maintenance. 

This is achieved by using the `runas` command with the `/noprofile` flag. The command below creates an elevated command line running under the `wunder\admin` user:

	runas /noprofile /user:wunder\admin cmd 

Now, all commands executed within this new command prompt would be in the same elevated status.

<!--excerpt-->

## References

- [Runas - Run under a different user account | Windows CMD | SS64.com](http://ss64.com/nt/runas.html)
- [User Account Control](http://windows.microsoft.com/en-us/windows7/products/features/user-account-control)
- [SuperUser - Is there any 'sudo' command for Windows?](http://superuser.com/questions/42537/is-there-any-sudo-command-for-windows)
- [StackOverflow](http://stackoverflow.com/questions/12903629/how-do-i-run-a-program-from-command-prompt-as-a-different-user-and-as-an-admin)