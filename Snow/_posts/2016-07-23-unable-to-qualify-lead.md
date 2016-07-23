---
layout: post
title: Unable to Qualify Lead
category: Dynamics CRM, Dynamics CRM Online
---
I was investigating an issue in a customized Dynamics CRM environment where the "Qualify" ribbon button on the Lead record had mysteriously stopped working. 

Qualification from the grid and the organization service API appeared to work as expected, and this only seemed limited to the ribbon button. Given that the page was not even posting back to the server on clicking the button, I suspected it was failing at the front end. I fired up the chrome developer tools and sure enough was greeted with the following error on the console:

![Developer Tools Error Message](/images/posts/UnableToQualifyLead/10_error-message.png)

<!--excerpt-->

I eventually found [this forum post](https://community.dynamics.com/crm/f/117/t/156598) that gave me the solution. It turns out that the out of the box JavaScript appears dependent upon two fields having to exist on the form - the `companyname` and `transactioncurrencyid` fields. They don't necessarily have to visible on the form, but added somewhere on the form.

This particular environment was running Dynamics CRM 2015 OnPremise with update `0.1`. My attempts at trying to replicate the issue on a CRM Online 2016 instance (version `8.1.0.367` at the time of writing) were unsuccessful. The qualification process worked fine even without the fields on the form. This leads me to believe that this dependency might have been removed post 0.1.


## References
- [Unable to Qualify Leads After 2015 Upgrade - Microsoft Dynamics CRM Community Forum](https://community.dynamics.com/crm/f/117/t/156598)