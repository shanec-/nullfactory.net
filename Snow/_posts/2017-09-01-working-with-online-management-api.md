---
layout: post
title:  Getting started with the Online Management API Sample
category: Dynamics CRM Online, 
published: draft
---

I've been wanting to play around with the Online Management API since the day it was announced. I finally got some time to put it through its paces.

I first wanted to get the sample application working.

Start off my registering your app and obtaining a client id and redirection Url.

https://msdn.microsoft.com/library/mt622431.aspx



I am going to re-use the authentication helper code in this case.

https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/sample-authentication-helper

The above code is dependent on the ActiveDirectory

https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory

https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/get-started-online-management-api#service-url

Looks like the latest version of the Microsoft.IdentityModel.Clients.ActiveDirectory library has been updated to use the async variant:

https://stackoverflow.com/questions/38012875/how-to-acquiretoken-with-promptbehavior-in-the-latest-microsoft-identitymodel-cl

Let's make some quick tweaks to the procedure to work with the Async method:

```c#
    /// <summary>
    /// Returns the authentication result for the configured authentication context.
    /// </summary>
    /// <returns>The refreshed access token.</returns>
    /// <remarks>Refresh the access token before every service call to avoid having to manage token expiration.</remarks>
    public AuthenticationResult AcquireToken()
    {
        //return _authContext.AcquireTokenAsync(_resource, _clientId, new Uri(_redirectUrl),
        //    PromptBehavior.Always).Result;
        var platformParameters = new PlatformParameters(PromptBehavior.Always);
        return _authContext.AcquireTokenAsync(_resource, _clientId, new Uri(_redirectUrl), platformParameters).Result;
    }
```
https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/7c9091a0edecf401fea402275e4a64aca95e40fe/src/ADAL.PCL.Desktop/PlatformParameters.cs





Here is a list of all operations supported by the Online Management API:

https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/operations-supported

The ones that I am quite excited about are the Create Instance and Delete Instances as these have potential to integrate nicely with my release strategy.

##  The Plan

This post is just outlining and getting a feel for the steps and actions necessary in order to perform the following steps:

1. Delete the new sandbox instance.
2. Create a new instance.

I intend the post to be a precursor to one that shows how we can integrate this functionality in order to complement an automated release strategy.

The samples in this post use the `Oceania` [service url](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/get-started-online-management-api#service-url). 

### Prerequisites

At the time of writing this post, the Online Management API [does not appear to support](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/operations-supported) the ability to convert a `Production` instance to a `Sandbox`. This is fine as I can do the conversion as a manual prerequisite step:

1. Register for a Trial instance.
2. Log into the `Dynamics 365 Administration Center` via the `Office Admin Center`. 
3. Convert the `Production` trial instance to a `Sandbox` instance. Do this by selecting the instance, clicking the `edit` button and selecting `Sandbox` in the `Instance type` drop down. 
4. Click the `Save` button when done.

## Delete the Instance

First, let's retrieve the instance identifier of using the [Get Instance](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/getinstance) operation:

	GET https://admin.services.crm6.dynamics.com/api/v1/instances

This returns us details about our instance:

    {
        "Id": "16d11c4e-1184-44c1-8742-8bfff382ad3a",
        "UniqueName": "orged5cac1e",
        "Version": "8.2.1.359",
        "ApplicationUrl": "https://sndxb16.crm6.dynamics.com/",
        "ApiUrl": "https://sndxb16.api.crm6.dynamics.com",
        "State": 1,
        "StateIsSupportedForDelete": true,
        "AdminMode": false,
        "Type": 2,
        "Purpose": null,
        "FriendlyName": "friendly",
        "DomainName": "sndxb16",
        "BaseLanguage": 1033,
        "InitialUserEmail": "shane.carvalho@invalid.com",
        "SecurityGroupId": "00000000-0000-0000-0000-000000000000"
    }

It was annoying trying to figure out what each of the `type` values meant, as the documentation does not explicitly state what each one maps to. Nor does the service provide a way to query this metadata. I had to infer the values by [switching the instance type in the admin center](https://technet.microsoft.com/library/dn896590.aspx) and then re-running the [Get Instance](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/getinstance) operation again.

Here are the values for `Production` and `Sandbox`:

| Instance Type | Type Value |
| ------------- | ---------- |
| Production    | 1          |
| Sandbox       | 2          |

Now that we have the unique handle (`16d11c4e-1184-44c1-8742-8bfff382ad3a`) to the organization instance, let's use it to delete it. Execute a [Delete Instance](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/deleteinstance) operation:

	DELETE https://admin.services.crm6.dynamics.com/api/v1/instances/16d11c4e-1184-44c1-8742-8bfff382ad3a/Delete

    {
        "OperationId": "51464381-9087-4b60-9069-3abba9f87bb7",
        "Status": 2,
        "Errors": [],
        "Information": [],
        "OperationLocation": "https://admin.services.crm6.dynamics.com/api/v1/Operation/51464381-9087-4b60-9069-3abba9f87bb7",
        "ResourceLocation": null,
        "Context": {
            "Items": {
                "InstanceState": 2,
                "admin.InstanceId": "16d11c4e-1184-44c1-8742-8bfff382ad3a"
            }
        }
    }

The response back gives a reference back to the operation in order to monitor the progress.

## Create a New Instance

The [creation of new instances](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/provisioninstance) is done using the `Provision Instance` call and one of the required parameters is the `ServiceVersionId`. This identifier is obtained via the  [Get Service Versions](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/getserviceversions) request: 

	GET https://admin.services.crm6.dynamics.com/api/v1/ServiceVersions

Unfortunately, I kept receiving a `HTTP/1.1 500 Internal Server Error` every time I made the request.

```json
{
    "message": "An error was encountered while processing your request. If problems persist, please contact customer support.",
    "time": "2017-09-02T04:40:42.9485913Z",
    "activityId": "2708135d-0066-40df-bede-24c173e0178b"
}
```

Not much information to work with, so I tried executing the same request using [Postman](https://www.getpostman.com/) and the request went through without a hitch. Okay, that means that there's nothing wrong with the service nor the actual URI that I was calling - which only leaves the headers.

Long story short, I setup fiddler as a proxy and compared the requests being made by my code against the one being made by Postman.

<< image of the fiddler request >>

It turns out that I needed to pass in the `Accept-Language` header as part of the request. Information about the required standard headers are [documented here](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/get-started-online-management-api#standard-headers).

At the time of writing, the [sample authentication code](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/authentication) did not include this by default, I've updated it to include the new header:

```c#
request.Headers.Add("Accept-Language", "en-US,en;q=0.8");
```
<< image of the updated code block >>

Note: The `q=0.8` value is the *relative quality factor* read more about it [here](https://stackoverflow.com/questions/8552927/what-is-q-0-5-in-accept-http-headers) and [here](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).

Now the service returns the expected information, including the service identifier (`Id`) we need for the provisioning request:

    [
      {
        "LocalizedName": "Dynamics 365",
        "LCID": 1033,
        "Version": "8.2",
        "Id": "31000000-c6b1-0000-x0x0-73x0x041bc5c",
        "Name": "Dynamics 365"
      }
    ]

Now let's kick off the creation of a new instance:

	PUT https://admin.services.crm6.dynamics.com/api/v1/instances/Provision/

    {
        "ServiceVersionId": "31000000-c6b1-0000-x0x0-73x0x041bc5c",
        "Type": "1",
        "FriendlyName": "superfriendly",
        "DomainName": "sndbx16",
        "BaseLanguage": "1033",
        "InitialUserEmail": "admin@sndbx16.onmicrosoft.com"
    }

The service returns confirmation that the process has been started:


    {
        "OperationId": "00000000-0000-0000-0000-000000000000",
        "Status": 6,
        "Errors": [],
        "Information": [],
        "OperationLocation": null,
        "ResourceLocation": "https://admin.services.crm6.dynamics.com/api/v1/Instances/16d11c4e-1184-44c1-8742-8bfff382ad3a",
        "Context": {
            "Items": {
                "admin.InstanceId": "16d11c4e-1184-44c1-8742-8bfff382ad3a",
                "InstanceState": 1
            }
        }
    }

Once again, neither the documentation nor the API describe what the `status` numbers mean. Hopefully the documentation will be updated to include this information.

You can test out the process yourself with the sample console application that I cobbled together available here.


## References

- [Operations supported by Online Management API for Dynamics 365 Customer Engagement | Microsoft Docs](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/operations-supported)
- [Get started with Online Management API for Dynamics 365 Customer Engagement | Microsoft Docs](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/get-started-online-management-api#service-url)
- [Postman | Supercharge your API workflow](https://www.getpostman.com/)
- [Authenticate to use the Online Management API for Dynamics 365 Customer Engagement | Microsoft Docs](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/authentication)
- [Provision Instance - Online Management API Reference | Microsoft Docs](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/provisioninstance)
- [Get Service Versions - Online Management API Reference | Microsoft Docs](https://docs.microsoft.com/en-us/rest/api/admin.services.crm.dynamics.com/getserviceversions)
- [TechNet - Switch an Instance](https://technet.microsoft.com/library/dn896590.aspx)
- [Standard Headers - Get started with Online Management API for Dynamics 365 Customer Engagement | Microsoft Docs](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/online-management-api/get-started-online-management-api#standard-headers)
- [What is q=0.5 in Accept* HTTP headers? - Stack Overflow](https://stackoverflow.com/questions/8552927/what-is-q-0-5-in-accept-http-headers)
- [HTTP/1.1: Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
- https://msdn.microsoft.com/en-us/library/gg327838.aspx