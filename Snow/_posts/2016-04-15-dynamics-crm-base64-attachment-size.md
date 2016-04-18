---
layout: post
title: Base64 and Maximum Attachment Size in Dynamics CRM
category: Dynamics CRM, Dynamics CRM Online
---
Microsoft Dynamics CRM provides out-of-the-box functionality to store and associate small file attachments against an entity record. This is achieved via the `annotation` entity - it is similar to any other entity except that it stores meta data as well as the [actual content of the file attachment](https://msdn.microsoft.com/en-us/library/gg334398.aspx) within it. 

As CRM is not designed to be a [file store](http://www.optimalcrm.co.uk/storage-limit-reached-in-microsoft-dynamics-crm-online/), it imposes some limitations on the size of attachments that can be uploaded. Recent versions of the product, including 2016, defaults this size to 5120Kb which can be adjusted to a [maximum hard limit of 32768Kb](https://dynamicscrmherald.wordpress.com/2014/01/03/increasing-web-resource-and-email-note-attachment-file-size-in-microsoft-dynamics-crm-2013/). 

However, there is one caveat with this limitation - CRM stores the byte stream in a base64 encoded format, this means that there would be an [increase](http://stackoverflow.com/questions/4715415/base64-what-is-the-worst-possible-increase-in-space-usage) in the final saved file size. 

<!--excerpt-->

I recently learned that the upload restriction is done based on the encoded file size and not the original byte stream as I was expecting. I imagine this would be especially important to remember when designing integration components that create `annotation` records.

## References
- [MSDN - Annotation (note) entity](https://msdn.microsoft.com/en-us/library/gg334398.aspx)
- [Storage Limit Reached in Microsoft Dynamics CRM Online](http://www.optimalcrm.co.uk/storage-limit-reached-in-microsoft-dynamics-crm-online/) 
- [Increasing Web Resource and Email / Note Attachment File Size in Microsoft Dynamics CRM 2013 | The Dynamics CRM Herald](https://dynamicscrmherald.wordpress.com/2014/01/03/increasing-web-resource-and-email-note-attachment-file-size-in-microsoft-dynamics-crm-2013/)