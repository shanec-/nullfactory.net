---
layout: post
title: Start Azure Web Jobs On Demand
category: Azure, Powershell, REST
---

I've been working on an Azure based solution recently and have been using the free tiers to quickly get the solution up and running and to perform the first few QA cycles. The core solution is based around single app service website and then a second website that acts as the host for a continuous web job which is triggered via a queue.
The problem with the free tiers is that there's a high possibility that the web job would shut itself down and hibernate if it is [idle for more that 20 mins](http://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/):

> As of March 2014, web apps in Free mode can time out after 20 minutes if there are no requests to the scm (deployment) site and the web app's portal is not open in Azure. Requests to the actual site will not reset this.

A little research shows a few possible solutions:

1. If the job is not time sensitive, then manually start the service remotely using a script or tool. 
2. Make your code explicitly start the web job just as a new request is being enqueued. This can be done by [making a REST call](https://github.com/projectkudu/kudu/wiki/WebJobs-API) to the deployment site.
3. And lastly, upgrade to a basic or standard tier and enabling "Always On" keeps the site (and jobs) "warm" and prevent them from hibernating.

<!--excerpt-->

### Using Powershell

This method requires the `Microsoft Azure Powershell` module to be installed on the machine. This can be [installed via the Web Platform Installer](http://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/).

1. Start of by logging into the Azure account and registering your subscription 
	
	`Add-AzureAccount`

2. Next, starts the job: 

	`start-azurewebsitejob -name samplejobweb -jobname sample-execution-job -jobtype continuous`
	
3. And this command to stop the job: 
	
	`stop-azurewebsitejob -name samplejobweb -jobname sample-execution-job`

### Using REST / Code

I [used this code as a template](http://stackoverflow.com/questions/28904186/how-can-i-keep-my-azure-webjob-running-without-always-on/28923039#28923039)  to whip up a console application that can be used to start the service:

	static void Main(string[] args)
	{
	    string websiteName = "<website name>";
	    string webjobName = "<job name>";
	    string userName = "<publishing username>";
	    string userPWD = "<publishing password>";
	    string webjobUrl = string.Format("https://{0}.scm.azurewebsites.net/api/continuouswebjobs/{1}", websiteName, webjobName);
	
	    var result = GetWebjobState(webjobUrl, userName, userPWD);
	    Console.WriteLine(result);
	
	    var postResult = StartWebjob(webjobUrl + "/Start", userName, userPWD);
	    Console.WriteLine(postResult);
	    
	    Console.ReadKey(true);
	}
	
	private static JObject GetWebjobState(string url, string username, string password)
	{
	    var client = CreateHttpClient(username, password);
	    var data = client.GetStringAsync(url).Result;
	    var result = JsonConvert.DeserializeObject(data) as JObject;
	    return result;
	}
	
	private static bool StartWebjob(string url, string username, string password)
	{
	    var client = CreateHttpClient(username, password);
	    HttpResponseMessage data = client.PostAsync(url, new StringContent(string.Empty)).Result;
	    return data.StatusCode == System.Net.HttpStatusCode.OK;
	}
	
	private static HttpClient CreateHttpClient(string username, string password)
	{
	    HttpClient client = new HttpClient();
	    string auth = "Basic " + Convert.ToBase64String(Encoding.UTF8.GetBytes(username + ':' + password));
	    client.DefaultRequestHeaders.Add("authorization", auth);
	    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
	    return client;
	}

## References
- [Run Background tasks with WebJobs](http://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/)
- [How can I keep my Azure WebJob running without "Always On" - Stack Overflow](http://stackoverflow.com/questions/28904186/how-can-i-keep-my-azure-webjob-running-without-always-on/28923039#28923039)
- [azure - continuous WebJob stops automatically - Stack Overflow](http://stackoverflow.com/questions/28502696/continuous-webjob-stops-automatically)
- [Github  - WebJobs API · projectkudu/kudu Wiki](https://github.com/projectkudu/kudu/wiki/WebJobs-API)
- [Github - Deployment credentials · projectkudu/kudu Wiki](https://github.com/projectkudu/kudu/wiki/Deployment-credentials)
- [Github - Azure/azure-powershell](https://github.com/Azure/azure-powershell)
- [MSDN - How to install and configure Azure PowerShell](http://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)
- [MSDN - Azure Cmdlet Reference](https://msdn.microsoft.com/library/azure/jj554330.aspx)
 

