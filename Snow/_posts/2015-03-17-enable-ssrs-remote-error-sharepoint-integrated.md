---
layout: post
title: Enable SSRS Remote Errors in SharePoint Integrated Mode
category: SharePoint, SQL Server Reporting Services
published: draft
---

I was working on some SQL Server Reporting Services (SSRS) reports today and ran into the usual error message of

For some reason which I could not figure out, the server would not recognize that I was on the local web server attempting to view the error message. 

1. Navigate to `Central Administration > Application Management > Manage Service Applications`
1. Next, click on the appropriate  `SQL Server Reporting Services Service Application` service application to manage it. 
1. Enable remote errors by navigating into `System Settings` and checking the `Enable Remote Errors` checkbox. 
