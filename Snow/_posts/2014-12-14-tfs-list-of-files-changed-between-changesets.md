---
layout: post
title: Get a list of files changed between changesets in Visual Studio Online using PowerShell
category: ALM, Team Foundation Server, PowerShell
---

So recently I had the requirement of getting a list of files that changed between two different releases. We wanted to use this list to act as a verification to ensure that all artifacts were included in a release package.

I modified the code posted [here](https://social.msdn.microsoft.com/Forums/vstudio/en-US/f1a00836-cef3-419b-b768-3d1b6fa2b7bc/identifying-all-vb-files-changed-between-two-changesets?forum=tfsversioncontrol) in order to quickly write a console application to do the task. With the immediate problem solved, my colleges and I bounced the idea about porting the code into a PowerShell script which would allow us to enhance it better in the long run.

The solution would be built around the Visual Studio Online(VSO) REST service. This reduces any dependency on Team Foundation Server(TFS) specific client side assemblies or tools. The limitation is that, at the moment, it is only supported in Visual Studio Online and not all features are supported.

### Pre-Requisites
#####Security and Credentials
In order make things simple, let's enable Alternate Authentication for access the account. This enables the script to use Basic Authentication when making request to the VSO REST service.
This can be done by navigating to the profile page, selecting `Credentials > Enable alternate credentials` and providing new credential information. More instructions available [here](http://www.visualstudio.com/en-us/integrate/get-started/get-started-auth-introduction-vsi).

The credentials will be collected using the `Get-Credentials` [cmdlet](http://technet.microsoft.com/en-us/library/hh849815.aspx). This provides the standard windows credentials dialog for the user to enter information. Since this makes the script interactive, I debated about having the username and password as a parameter for the script, but in the end decided against it. Maybe the next improvement would be to include a silent version of the script.


#####The REST call
`Invoke-RestMethod` cmdlet will be used to make the actual call to the REST service. So what's the difference between `Invoke-WebRequest` and `Invoke-RestMethod` you may ask? While similar, the `Invoke-RestMethod` attempts to parse the returned JSON so that we do not have to do it manually within our script. Think of it as a super set of `Invoke-WebRequest` just like `Invoke-WebRequest` is a superset of `System.Net.WebClient`. Read more about it [here](http://jamesone111.wordpress.com/2014/06/09/screen-scraping-for-pleasure-or-profit-with-powershells-invoke-restmethod/) and [here](http://blogs.technet.com/b/heyscriptingguy/archive/2013/10/21/invokerestmethod-for-the-rest-of-us.aspx).

I ran into strange issue when attempting to authenticate the request. The  
`Get-Credentials` cmdlet would return a `System.Management.Automation.PSCredential` object as expected, but when passed into into the `Invoke-RestMethod` cmdlet, it was not generating the the basic authentication header token within the request. I still haven't figured out why this happens, but the workaround was to add the authentication header explicitly as [shown here](http://stuartpreston.net/2014/05/accessing-visual-studio-online-rest-api-using-powershell-4-0-invoke-restmethod-and-alternate-credentials/). 

    $basicAuth = ("{0}:{1}" -f $username,$password)
    $basicAuth = [System.Text.Encoding]::UTF8.GetBytes($basicAuth)
    $basicAuth = [System.Convert]::ToBase64String($basicAuth)
    $headers = @{Authorization=("Basic {0}" -f $basicAuth)}

###Making the call to the service

1. First get a list of changesets related to the project within the timeframe that we're interested in.  

		https://{account}.visualstudio.com/defaultcollection/_apis/tfvc/changesets?api-version=1.0&searchCriteria.fromId=100&searchCriteria.toId=200&searchCriteria.itemPath=$/{project}

	It took me a while to figure it out but you should notice this call is only allowed to be made against the entire Team Project Collection. So in order to filter out the project, provide the project path via the `searchCriteria.itemPath` filter. That is `searchCriteria.itemPath=$/{projectname}` where `{projectname}` is the one that you are interested in.
1.  Next iterate through each of the results to retrieve the detailed information on of each of the changesets. This result would include a collection of all the files that were affected.
	
		https://{account}.visualstudio.com/defaultcollection/_apis/tfvc/changesets/{changesetId}/changes?api-version=1.0

1. Again, iterate through each of the changes and extract the `path` property of the json result set. This is the path and name of the file.
1. Remove duplicates entries and folder creation entries as necessary.   


###Final Thoughts
You can download my implementation [here](https://github.com/shanec-/powershell/blob/master/TFS/Get-FilesModifiedByChangeset.ps1).

The next steps would make this to its own cmdlet in order to make it more reusable in other scripts.
Also check out the [Curah! page](https://curah.microsoft.com/276618/list) that I created while working on this. 

###References
- [Identifying all .vb files changed between two changesets](https://social.msdn.microsoft.com/Forums/vstudio/en-US/f1a00836-cef3-419b-b768-3d1b6fa2b7bc/identifying-all-vb-files-changed-between-two-changesets?forum=tfsversioncontrol)
- [Microsoft Technet - Get-Credential](http://technet.microsoft.com/en-us/library/hh849815.aspx)
- [Screen scraping for pleasure or profit (with PowerShell’s Invoke-RestMethod)](http://jamesone111.wordpress.com/2014/06/09/screen-scraping-for-pleasure-or-profit-with-powershells-invoke-restmethod/)
- [Hey, Scripting Guy! - InvokeRestMethod for the Rest of Us](http://blogs.technet.com/b/heyscriptingguy/archive/2013/10/21/invokerestmethod-for-the-rest-of-us.aspx)
- [Microsoft Technet - Invoke-RestMethod](http://technet.microsoft.com/en-us/library/hh849971.aspx)
- [Accessing Visual Studio Online REST API using Powershell 4.0, Invoke-RestMethod and Alternate Credentials](http://stuartpreston.net/2014/05/accessing-visual-studio-online-rest-api-using-powershell-4-0-invoke-restmethod-and-alternate-credentials/)