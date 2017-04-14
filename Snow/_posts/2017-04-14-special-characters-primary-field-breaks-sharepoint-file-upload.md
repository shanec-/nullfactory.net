---
layout: post
title: Special Characters in Primary Field Breaks SharePoint File Upload in Dynamics 365
category: Dynamics CRM, Dynamics CRM Online
---
My colleagues and I ran into this confusing issue the other day. We've been working on a Dynamics 365 Online instance integrated with SharePoint (as a server side configuration) and everything seem to be working as expected but for a few records. On these records, anytime the user hits the upload button in document management they are greeted with the following error message instead of the expected upload dialog:

> An error has occurred. 
> 
> Try this action again. If the problem continues, check the Microsoft Dynamics 365 Community for solutions or contact your organization's Microsoft Dynamics 365 Administrator. Finally, you can contact Microsoft Support.

![Upload Error Dialog](/images/posts/BrokenSharePointInteg/10_error_dialog.png)

<!--excerpt-->

When I navigated to the physical location on the SharePoint server, I can see that a folder record was created successfully in the Document Library and that the logged user has permission to make changes to the folder and its contents. 

Next, I tried manually uploading a document through the SharePoint server and in order to verify that the document was reflected back in the CRM integration component. This too was successful and I verified that it has the ability to perform CRUD operations as expected. 

None of these seemed to get me closer to identifying the issue - the upload control in CRM was still broken for the record.

![SharePoint Folder](/images/posts/BrokenSharePointInteg/20_sharepointfolder.png)

## The Solution

With a little bit of trial and error we were finally able to narrow down the issue - it turned out to be the value of the primary field of the record. 

By default, the integration component uses the value of the primary field in order to generate a unique folder name in SharePoint, it takes parts of the name and then appends a unique salt to the end. 

So the problem in our broken record set was that the primary field value contained special characters. While the folder creation itself appears to have cleaned out the these characters, the upload control still did not like it. 

![SharePoint folder invalid characters](/images/posts/BrokenSharePointInteg/30_folder_invalid_chars.png)

Therefore, the fix itself pretty simple - remove the invalid characters from the primary field and the upload control started working as expected.

![valid SharePoint Folder](/images/posts/BrokenSharePointInteg/40_validfolder.png)

This issue appears to exists as of `Microsoft Dynamics 365 Version 1612 (8.2.1.164) (DB 8.2.1.164) online`.

I can think of two ways we can avoid this in the future:

- Add validation on the primary field to explicitly filter out the special characters - this is the most sensible approach until Microsoft fixes the upload control.
- Handle the creation of the SharePointLocation record and the physical SharePoint folder yourself - this is obviously more work, but provides you more granular control.

I hope this helps anyone else who might encounter this issue.