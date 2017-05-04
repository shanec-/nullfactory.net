---
layout: post
title: Reassignment of Record Causes Attached Notes to Update its ModifiedBy User Field in Dynamics CRM
category: Dynamics CRM, Dynamics CRM Online
---
My colleagues and I ran into a interesting problem the other day; we noticed that on some records all the notes attached to it were being updated seemingly randomly being updated. At first glance I was confused as to why this was happening, but we eventually found the steps to reproduce it: 

1. Create a record `Record 1` in an entity. This can either be a custom entity or out-of-the-box entity with notes and attachments enabled. 
1. If the notes control has not been added onto the form, do that now.
1. Update it such that it has several notes created by different users.
1. Next make sure that `Record 1` is owned by `User A`.
1. Login as `User B` and reassign the `Record 1` to `User C`.

Now, if we observe the notes control it shows `User B`'s name along side the date and time stamp. 

I was initially thrown off as to why the above actions changed the `Created By` of the notes, but what I failed to realize was that the annotation control shows the `Modified By` user and not the `Created By`.
 
<!--excerpt-->

The relationship between the custom entity and the `annotation` entity is parental with `cascade all` set on assignment, therefore when the parent record is reassigned, it automatically reassigns all the notes associated with it. The reassignment itself is considered an update and thus shows the the user who initiated the reassign as the `Modified By` user.  And if you think about it you would see that CRM is behaving exactly as it should - the parent record and the notes are assigned to `User C` while the last modified is set to `User B`.

In this particular custom solution I was working on, the notes were being treated as immutable - that is the security roles were structured such that once created it would not be modified. The reassignment logic is done by an external process executed under an service account. So the reassignment and subsequent change of the `Modified By` to the service account was confusing the users. 

I found that there are two potential workarounds for this, but each with its own caveats.

## Workaround 1

The first approach is to alter the parental relationship of the record. Change the relationship behavior from `Parental` to `Configurable Cascading` and then the `Assign` option from `Cascade All` to `Cascade None`.

![Updated Parental Relationship](/images/posts/ReassignChildNotes/10_updated_relationship.png)

One caveat of this approach is that if the parent record is reassigned from a user in a higher business unit hierarchy to one in a lower business unit, the notes would not be visible to other users in the lower business unit. This is of course subject to the way the security permissions have been configured - your mileage may vary.

## Workaround 2

Write a `pre-operation` plugin that runs on the update of the annotation entity. Have it configured to run on the same service account as the one used by the external process. This way when the record is reassigned by the external process the last modified by user is still retained. 

Read more this approach [here](http://missdynamicscrm.blogspot.com.au/2014/06/crm-2011-2013-modify-createdon-createdby-modifiedon-modifiedby-using-SDK-CSharp.html).

    public class RetainModifiedByUserPlugin : IPlugin
    {
    	public void Execute(IServiceProvider serviceProvider)
    	{
    		IPluginExecutionContext context =
    			(IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
    
    		#warning todo: defensive programming!
    
    		var preImage = context.PreEntityImages["Image"];
    		var previousUser = (EntityReference)preImage["modifiedby"];
    
    		var updateRequest = new UpdateRequest() { Parameters = context.InputParameters };
    		var modifiedBy = (EntityReference)updateRequest.Target["modifiedby"];
    
    		// if record is modified by the service account
    		if (modifiedBy.Id == context.UserId)
    		{
    			updateRequest.Target["modifiedby"] = previousUser;
    		}
    	}
    }

The downside to this approach is that updating user has to be the designated "service account". I also think that changing the expected behaviour of the update functionality is not the best user experience, but once again depending on the requirement your mileage may vary. 

I can also confirm that this solution still works even with the latest version of Dynamics 365 online (`Version 1612 (8.2.1.176) (DB 8.2.1.176) online` as of writing this post).

## References

- [CRM 2011/ 2013 Modify CreatedOn, CreatedBy, ModifiedOn, and ModifiedBy Using SDK C# ~ Ms. Dynamics CRM](http://missdynamicscrm.blogspot.com.au/2014/06/crm-2011-2013-modify-createdon-createdby-modifiedon-modifiedby-using-SDK-CSharp.html)
- [The truth about ‘Override Created on or Created by for records during data import’ – EMEA Dynamics CRM Support](https://blogs.msdn.microsoft.com/emeadcrmsupport/2012/08/01/the-truth-about-override-created-on-or-created-by-for-records-during-data-import/)
- [The truth about ‘Override Created on or Created by for records during data import’ – EMEA Dynamics CRM Support](https://blogs.msdn.microsoft.com/emeadcrmsupport/2012/08/01/the-truth-about-override-created-on-or-created-by-for-records-during-data-import/)