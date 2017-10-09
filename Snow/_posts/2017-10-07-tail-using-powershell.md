---
layout: post
title: Tail using PowerShell
category: PowerShell
---
Consider a scenario where you are diagnosing an issue on a production server. You've enabled the logs and got everything setup to reproduce the issue. Queue the trigger to invoke the misbehaving operation and finally wait for the log file to update.

Staring at the log file size to change is no fun nor is refreshing the file periodically. I want to be notified the moment the log gets updated in real-time. This is exactly what a program like [`tail`](https://en.wikipedia.org/wiki/Tail_(Unix)) does - it monitors the file for changes and outputs the *tail* end of the file.

There are a few incarnations of tail out there that do the job perfectly fine, but given that this was a production environment I was not keep about installing new tools.

Luckily, the `Get-Content` PowerShell cmdlet has this functionality built-in. This is the combo that I finally settled on:

<!--excerpt-->

`Get-Content log.txt -tail 5 -wait`

The `-tail` argument takes in the number of lines to show from the end of the file and `-wait` specifies that  it should continue to monitor the file.

Granted that this method does not have highlighting capabilities that a dedicated tool would have, but its a pretty useful option to have in a pinch.

Finally, save a few key strokes by using the alias:

`cat log.txt -tail 5 -wait`

## References

- [Looking for a windows equivalent of the unix tail command - Stack Overflow](https://stackoverflow.com/questions/187587/looking-for-a-windows-equivalent-of-the-unix-tail-command)
- [13 Ways to Tail a Log File on Windows & Linux: Top Tools](https://stackify.com/13-ways-to-tail-a-log-file-on-windows-unix/)
- [Microsoft Docs - Get-Content](https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Management/Get-Content?view=powershell-5.1)

