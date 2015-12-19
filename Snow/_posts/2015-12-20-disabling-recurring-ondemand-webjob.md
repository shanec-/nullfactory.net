---
layout: post
title: Disabling Recurring and OnDemand Web Jobs within a Deployment Slot 
category: Azure
---

I've come to realize that things can get a bit tricky when working with slot deployments and webjobs. I learned *the hard way* that stopping a slotted Web App does **not** stop the web jobs it hosts. This means that I can't just stop all my website slots and expect the related jobs to automatically shutdown as well. Bummer. 
Okay, so I am thinking maybe I'll just disable the individual jobs for each of the slots? Not much luck on that front either as the Azure portal only provides a `stop` option for `continuous` jobs and not for the `recurring` and `OnDemand` jobs.

This limits us to a few possible solutions:

<!--excerpt-->

1. Make sure that all the resources accessed by the slotted web app / jobs are isolated from each other - Ensure that the necessary appsettings are defined as slot settings and point to different values. This is done so as to ensure that you do not inadvertently run a scheduled job multiple times (once via the production slot and one time for each of the slots). 
2. Baking-in custom disabling logic right into your job - For example, a web job could skip its operation based on a custom appsetting value. Once again, this appsetting would be defined as a slot setting. 
	
	> A word of caution - be weary when implementing the "skipping" logic on continous jobs as "skipping" can be considered a successful run which would in turn pop the last message in the tiggered queue.
  
3. The nuke option - entirely delete the web job entries in each of the slots (not recommended).