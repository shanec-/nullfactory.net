---
layout: post
title: Non-searchable Primary Key Field Breaks Pre-filtering
category: Dynamics CRM, Dynamics CRM Online
---
I recently implemented a custom FetchXml based SSRS report that was designed to be run against a filtered list of accounts. The accounts were to be determined by the user at the time of execution, hence the report was setup to use pre-filtering on the accounts entity.

The report worked fine except in one environment where the user was greeted with the following warning message:

![Filtering error](/images/posts/PreFilterNonSearchable/10_FilteringError.png)

Dismissing the warning opened up the `Report Filtering Criteria` dialog with the following error:

	There was an error in showing this condition

And additional information:

	The condition was referencing the field accountid. The field has been removed from the system or is not valid for advanced find.

![Report filtering criteria](/images/posts/PreFilterNonSearchable/20_ReportFilteringCriteria.png)

<!--excerpt-->

A quick bit of investigating revealed that this particular environment had the `Searchable` property of the primary key, within the account entity, set to `No`.
This meant that the `Account` attribute would not appear on any advanced find dialogs, there by affecting the `Reporting Filtering Criteria` dialog. Re-enabling this got the report working as expected.

## Reference
- [Create or edit entity fields | Microsoft Dynamics CRM](https://www.microsoft.com/en-us/dynamics/crm-customer-center/create-or-edit-entity-fields.aspx)