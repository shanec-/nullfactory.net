---
layout: post
title: Enable SSRS Remote Errors in SharePoint Integrated Mode
category: SharePoint, SQL Server Reporting Services
published: draft
---

I've been troubleshooting some SQL Server Reporting Services (SSRS) reports that were deployed in SharePoint-Integrated mode. I was seeing the usual familiar error message:

So as suggested, I tried logging into one of the web front ends (WFE) in order to see the underlying error but SSRS would not recognize that I was accessing the reports from the local server. I tried the other WFEs as well as the SQL Server itself, but nothing seemed to work. After an frustrating hour of trying to figure this out, I bit the bullet and decided to enable remote errors temporarily. 

Remote errors are enabled via the SSRS Service Application:

1. Navigate to `Central Administration > Application Management > Manage Service Applications`
1. Next, click on the appropriate  `SQL Server Reporting Services Service Application` service application to manage it. 
1. Enable remote errors by navigating into `System Settings` and checking the `Enable Remote Errors` checkbox. 

Although the MSDN documentation says this is all that is required, I had to enable it on the individual site's settings in order to get it to work:

1. Navigate to `Site Settings > Reporting Services > Reporting Services Site Settings`
2. Ensure that the `Enable Local Mode Error Messages` is checked


## References

- [StackOverflow - SSRS Remote Errors Enabled but NOT Working](http://stackoverflow.com/questions/4850346/ssrs-remote-errors-enabled-but-not-working)