---
layout: post
title: Resolving a Race Condition Within a Dynamics 365 Process
category: Dynamics CRM, Dynamics CRM Online
---

I was attempting to diagnose and issue for a client the other day. It revolves around a single workflow that is being executed on both the create and update event of a field. And within this workflow, a custom activity executed if the field matches a certain condition. The problem was that the custom activity was being executed twice when it was only expected to run once.

![Single Workflow](/images/posts/AsyncWorkflow/10_singleworkflow.png) 

![Workflow Triggger](/images/posts/AsyncWorkflow/20_singleworkflow_trigger.png)

If we take the above example, it can be replicated easily:

1. Create a new record where `Delivery Status` is `Current`.
1. Immediately update the record to `Pending`.

Navigate to the background processes and ensure that the workflow was executed twice, once on the create and then again on the update. If you performed the update soon enough, you notice that both instances satisfied the citieria and the custom activity invoked.

This behaviour is by design - when an async workflow is triggered it neither guarantees the immediate execution of the process nor the state of the record at the time it was queued. This means that the workflow would work against the latest version of the record at the time of execution.

<!--excerpt-->

## The Solution

The solution was to break the workflow process into two different parent-child workflows. The parent workflow evaluates the critiera and execute the child workflow, and the child workflow invokes the custom workflow activity. In order to avoid the same race condition we encountered earlier, the parent workflow is set to run `synchronously`. The child workflow will continue to be executed asynchronously.  

The new structure of the workflow:

![Split Synchronized Parent](/images/posts/AsyncWorkflow/30_SplitSyncParent.png) 

![Split Asynchronized Child](/images/posts/AsyncWorkflow/40_SplitAsyncChild.png) 

## Final Thoughts

I think the lesson learned here is that even though the benifits of going asynchronous are numerous, it is important to evaluate and implement one that works well with the particular business secenario. 