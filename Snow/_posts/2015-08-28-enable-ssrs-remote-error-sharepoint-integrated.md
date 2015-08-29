---
layout: post
title: Enable SSRS Remote Errors in SharePoint Integrated Mode
category: SharePoint, SQL Server Reporting Services
---

Any time I have to troubleshoot issues in SQL Server Reporting Services (SSRS) reports in a production environment, I usually end up enabling `Remote Errors` at some point as part my process. 

Remote errors are enabled via the SSRS Service Application:

1. Navigate to `Central Administration > Application Management > Manage Service Applications`
1. Next, click on the appropriate  `SQL Server Reporting Services Service Application` service application to manage it. 
1. Click `System Settings` from the toolbar.
1. Finally, enable remote errors by navigating into checking the `Enable Remote Errors` checkbox. 

<!--excerpt-->

Although the [MSDN documentation](https://msdn.microsoft.com/en-us/library/aa337165.aspx) states that this is all that is required, I've found that I need to enable it on the individual site's settings in order to get it working:

1. Navigate to `Site Settings > Reporting Services > Reporting Services Site Settings`
2. Ensure that the `Enable Local Mode Error Messages` is checked.

Once the troubleshooting session is over, follow the same steps in reverse order to disable remote errors.  

## References

- [Enable Remote Errors (Reporting Services)](https://msdn.microsoft.com/en-us/library/aa337165.aspx)
- [StackOverflow - SSRS Remote Errors Enabled but NOT Working](http://stackoverflow.com/questions/4850346/ssrs-remote-errors-enabled-but-not-working)
