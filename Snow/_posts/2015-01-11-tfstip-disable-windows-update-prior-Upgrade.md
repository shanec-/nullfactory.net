---
layout: post
title: TFS Tip: Disable Windows Update Prior TFS Upgrade
category: ALM, Team Foundation Server, TFS Tips
---
Always make sure to disable the windows update services before starting any significant software upgrade. We found out the hard way that process such as a long running database backup does not handle kindly to spontaneous server restarts.

Its one of those thing that can easily slip through the cracks but have a large impact in your deployment. So always make sure to add this to part of your per-requisite checklist.

Personally I use the `net stop "windows update"` command as it is only effective until you restart the computer/service.

<!--excerpt--> 

Here are some ways to achieve the same: 

* [http://superuser.com/questions/603315/how-do-i-stop-the-windows-8-restart-clock-15-minutes-and-counting](http://superuser.com/questions/603315/how-do-i-stop-the-windows-8-restart-clock-15-minutes-and-counting)
* [http://www.wikihow.com/Disable-Automatic-Reboot-After-Windows-Update](http://www.wikihow.com/Disable-Automatic-Reboot-After-Windows-Update)

## References

* [Super User - shutdown - How do I stop the Windows 8 restart clock; '15 minutes' and counting...?](http://superuser.com/questions/603315/how-do-i-stop-the-windows-8-restart-clock-15-minutes-and-counting)
* [WikiHow - 3 Ways to Disable Automatic Reboot After Windows Update](http://www.wikihow.com/Disable-Automatic-Reboot-After-Windows-Update)