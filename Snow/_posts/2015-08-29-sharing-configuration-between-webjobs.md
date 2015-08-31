---
layout: post
title: Sharing Configuration Between WebJobs
category: Azure
---

The project I am working on started out with the single webjob and has since grown to multiple jobs running in parallel. They are all hosted within a dedicated web app, which allows us to scale the jobs independent of the rest of the application. And because they share a single container there is the added side effect of the jobs sharing all the Application Settings and connection strings too.

While each webjob had its own class library, I didn't want to maintain multiple copies of  the `App.Config` file. I decided to share the the common bits (`AppSettings` and `ConnectionString` sections) in their own files:

1. In one of the webjob projects, I moved the `AppSettings` and `ConnectionStrings` into their own `.config` files - `appSettings.config` and `connectionStrings.config` respectively.

2. Next, I referenced them back to the `App.config` using the `configSource` attribute.

3. Finally, I added the same files as linked files to the otherweb jobs and set their `Copy to Output Directory` file property to `Copy Always`.

This works well enough, but for one caveat - which prompted me to write this post in the first place. The problem is that the `Web Deploy Package` publishing process does not appear to honor folder structures for the config files. That means that if you've separated the configuration into sub folders (like shown below), the publishing process would flatten it out.

<!--excerpt-->

	<appSettings configSource="CommonConfig/appSettings.config" />
	<connectionStrings configSource="CommonConfig/connectionString.config" />

![Solution Structure](/images/posts/ShareConfigWebJob/10_SolutionStructure.png)

This is the new structure created when published:

![Package Structure](/images/posts/ShareConfigWebJob/20_PackageStructure.png)

I suppose we could possibly make it work if we create the webjob folder structure manually in the App_Data folder in your web project, but I don't think its worth the additional complexity for such a trivial issue.

I guess the work around/best practice would be use a simple flat configuration folder structure.

## References

- [c# - How to use partial config files - Stack Overflow](http://stackoverflow.com/questions/7417062/how-to-use-partial-config-files)