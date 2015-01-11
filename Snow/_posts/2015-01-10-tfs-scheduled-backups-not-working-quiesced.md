---
layout: post
title: TFS Tip: Scheduled Backups Do Not Work While the Service Is Quiesced
category: ALM, Team Foundation Server, TFS Tips
---
Last month I worked on upgrading a Team Foundation Server from 2012.2 to 2013.4. While majority of the process was based on the off of the [ALM Rangers upgrade guide](http://vsarupgradeguide.codeplex.com/), there were a few interesting tidbits that we learned during the upgrade.  

The most important one was that Scheduled Backups feature no longer works if the services are stopped using the `TFSServiceControl quiesce` command.

While the [MSDN Article](http://msdn.microsoft.com/en-us/library/ff470382.aspx) for the command does state that it would take down all the services, it also says that you would normally use this command in order to facilitate backups. 
As such we assumed, incorrectly, that it would not apply to the scheduled backup service. 

<!--excerpt--> 

> The lesson learned here is that we should `quiesce` the service only for manual backups of the databases. 

One we figured this out, we manually took the backups and everything was alright in the world again. 

## References

* [MSDN - Understanding backing up Team Foundation Server](http://msdn.microsoft.com/en-us/library/ms253151.aspx)
* [Grant Holliday’s blog - TFS2012: What are all the different Jobs built-in to TFS?](http://blogs.msdn.com/b/granth/archive/2013/02/13/tfs2012-what-are-all-the-different-jobs-built-in-to-tfs.aspx)
* [CodePlex - TFS Upgrade Guide](http://vsarupgradeguide.codeplex.com/)
* [Chaminda's Blog - TFS Backup/Restore – Lessons Learnt](http://chamindac.blogspot.com/2014/12/tfs-backuprestore-lessons-learnt.html)