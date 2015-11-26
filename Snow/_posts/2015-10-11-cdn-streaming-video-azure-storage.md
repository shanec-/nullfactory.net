---
layout: post
title: Setting up a CDN to Stream Video via Azure Storage
category: Azure, CDN, Azure Storage Accounts
---
I needed to setup an Azure Content Delivery Network (CDN) in order to stream some video files stored in Azure Blob Storage. Sounds simple enough right? Well, yes for the most part, but I did hit a few hurdles along the way. This post would hopefully help me avoid them the next time.

## Create the Storage Account

Something I found out after the fact was that CDN endpoints [currently only support classic storage accounts](http://stackoverflow.com/questions/32569564/azure-resource-manager-deployment-vs-classic-deployment-of-storage-accounts). So the first order of business is to create a classic storage account either via old portal or using a [resource group manager template](/2015/10/deploy-classic-storage-azure-resource-manager/). 

Another thing I found out is that, at the time of writing, classic storage accounts cannot be made under the 'East US' location. The closest alternative was 'East US 2' and worked fine; I guess its something worth considering if you wanted to co-locate all your resources.

Next, create a container within storage account - the container would host the files that would be served by the CDN. It can be created manually via the old portal or even through visual studio. Ensure that container access type is set to `Public Blob`.

## Upgrade the Storage Account to a Newer Service Version 

The first time I tried to tried to stream a video, it did not work as expected; stream was very choppy. It turns out that the service version that got set on the storage was not the latest. [Read more here](http://blog.thoughtstuff.co.uk/2014/01/streaming-mp4-video-files-in-azure-storage-containers-blob-storage/), [and here](https://msdn.microsoft.com/library/azure/dd894041.aspx).

So the next step is update the storage account to the latest version in order to take advantage of the improvements. This can be done using the following code:

		var credentials = new StorageCredentials("accountname", "accountkey");
		var account = new CloudStorageAccount(credentials, true);
		var client = account.CreateCloudBlobClient();
		var properties = client.GetServiceProperties();
		properties.DefaultServiceVersion = "2013-08-15";
		client.SetServiceProperties(properties);
		Console.WriteLine(properties.DefaultServiceVersion);

<!--excerpt-->

## Create the CDN Endpoint

Setting up the CDN itself it pretty straight forward:

1. Create a new CDN through the old portal by selecting `New > CDN > Quick Create`. 
2. Select your subscription and set the origin type as `Storage Account`.

	[![Azure Create CDN](/images/posts/AzureCDNStream/10_CreateCDN.png)](/images/posts/AzureCDNStream/10_CreateCDN.png)

3. Select one of the classic storage accounts from the `Origin Url` drop down and hit the create button.

	[![Azure CDN Created](/images/posts/AzureCDNStream/20_CDNCreated.png)](/images/posts/AzureCDNStream/20_CDNCreated.png)

## Upload Content

Now that everything is setup, go ahead and upload the content into blob storage using Visual Studio or [Azure Storage Explorer](https://azurestorageexplorer.codeplex.com/). Once the content is propagated, video streaming should be smooth and working as expected.

## References

- [Stack Overflow - Azure Resource Manager Deployment vs Classic Deployment of Storage Accounts](http://stackoverflow.com/questions/32569564/azure-resource-manager-deployment-vs-classic-deployment-of-storage-accounts)
- [Stack Overflow - Is Microsoft Azure CDN A Real CDN Or Something Else Entirely?](http://stackoverflow.com/questions/7235082/is-microsoft-azure-cdn-a-real-cdn-or-something-else-entirely)
- [Streaming MP4 video in Azure Storage containers (Blob Storage) | thoughtstuff | Tom Morgan](http://blog.thoughtstuff.co.uk/2014/01/streaming-mp4-video-files-in-azure-storage-containers-blob-storage/)
- [MSDN - Versioning for the Azure Storage Services](https://msdn.microsoft.com/library/azure/dd894041.aspx)
- [Azure Storage Explorer - Home](https://azurestorageexplorer.codeplex.com/)


