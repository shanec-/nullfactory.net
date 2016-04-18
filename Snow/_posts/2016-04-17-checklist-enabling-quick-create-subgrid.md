---
layout: post
title: Checklist for Enabling Quick Create from Subgrid 
category: Dynamics CRM, Dynamics CRM Online
---
There have been a few instances where I've wanted to launch a quick create a record directly from a sub-grid add-button. That is, without having an extra step of bringing down the lookup view and then clicking the `+ New` button. 

![The expected result](/images/posts/QuickCreateSubGrid/10_ExpectedResult.png)

While the required steps are covered between these two great posts - [here](http://ledgeviewpartners.com/blog/manage-quick-create-forms-dynamics-crm/) and [here](http://www.powerobjects.com/2015/03/10/open-quick-create-sub-grid/), I have summarized the checklist here for convenience.

<!--excerpt-->

In order to get the CRM sub-grid to show the quick create dialog, ensure that:

1. The `Allow quick create` is enabled for the child entity.

    ![Enable Quick Create](/images/posts/QuickCreateSubGrid/20_AllowQuickCreateOption.png)

2. A quick create form exists for the entity. If not, create a new one.
    
    ![Ensure Quick Create form exists](/images/posts/QuickCreateSubGrid/30_EnsureQuickCreateForm.png)
    
3. Ensure that the related foreign key field is `Business Required`.

    ![Ensure foreign key is business required](/images/posts/QuickCreateSubGrid/40_BusinessRequired.png)

## References
- [How to Manage Quick Create Forms in Dynamics CRM - Ledgeview Partners](http://ledgeviewpartners.com/blog/manage-quick-create-forms-dynamics-crm/)
- [How to: Open Quick Create from a Sub Grid](http://www.powerobjects.com/2015/03/10/open-quick-create-sub-grid/)