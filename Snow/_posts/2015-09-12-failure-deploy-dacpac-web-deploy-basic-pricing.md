---
layout: post
title: Failure to Deploy a dacpac using Web Deploy when using a Basic Pricing Tier
category: Azure, Deployment
published: draft
---

The project I am working on uses WebDeploy in order to publish both the web application and database tier. 



After much trial and error and hair pulling, I narrowed it down to the pricing tier for the SQL Server. The problem turned out to be that we were originally using the `web` pricing tier. And given its imminent retirement, I had decided to upgrade to tier to the. I choose the `Basic` tier as the substitute.

I ran into this strange issue today that had left me scratching my head.

I was re-builing an Azure development environment that included a web app and a couple of SQL Server Databases. I use WebDeploy to deploy the web apps and


I was using a Azure Resource Template to do the deployment and had done so many time using the template, so was pretty certain that it was not the template that was at fault.

 

The gist of the issue is that if you've got a SQL Database running in a


The Workaround?

 


## References

- http://www.microsoft.com/en-us/download/details.aspx?id=43717