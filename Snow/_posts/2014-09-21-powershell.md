---
layout: post
title: Batch extract YouTube direct video url via Powershell
category: Powershell, HtmlAgilityPack
---
It's been a while since I've have had the opportunity to write any powershell. So I decided to do something before my muscle memory atrophies completely. 

While I could not come up with a new problem to solve, I decided to re-visit my attempt at extracting the direct download url from a streaming service.I believe this is my 3rd attempt and its fast becoming my version of a "hello world" application. :)

Previously, I tried to implement and parse out the urls by myself. This version is not going to be as complex as I will be using the [keepvid.com](http://www.keepvid.com) to do the heavy lifting. 

Here is a list of the steps:

1. Execute the script by either providing a space delimited list of video urls or location to a file containing the list of urls (one url per line).
> .\Get-DirectVideoUrl.ps1 -url -filename "C:\downloadlist.txt"
>
> .\Get-DirectVideoUrl.ps1 -url "http://www.youtube.com/watch?v=duKL2dAJN6I http://www.youtube.com/watch?v=R4ajQ-foj2Q"
>
> .\Get-DirectVideoUrl.ps1 -url "http://www.youtube.com/watch?v=duKL2dAJN6I http://www.youtube.com/watch?v=R4ajQ-foj2Q" -filename "C:\downloadlist.txt"

2. The script formats the url appropriate for the service and sends out the request.
3. Once the response is received, the html is parsed out using XPath with the help of the HtmlAgilityPack library. (while this could have been done without the additional dependency, I opted to use it in the hopes that future enhancements would be easier)
4. Repeat the process for all the input video urls that have been provided.
5. Save all the urls to a text file.

Now that you have the final urls, it can batch imported into a download accelerator such as [FDM](www.freedownloadmanager.org/).

While I only tested the script against the YouTube service I am sure this should work with any of the other video services supported by keepvid.com

The script is hosted [here](https://github.com/shanec-/powershell/tree/master/Get-DirectVideoUrl), feel free to push back any patches if you decide to improve on the code.

Same disclaimer applies to this post as per my previous attempts. This was written for educational purposes and I am pretty sure that it is violating the TOS video streaming service or keepvid itself. Therefore please use at your own discretion.