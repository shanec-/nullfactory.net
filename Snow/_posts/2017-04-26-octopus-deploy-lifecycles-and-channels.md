---
layout: post
title: Creating a Custom Lifecycle and Channel for Octopus Deploy Deployment 
category: Octopus Deploy, ALM
published: draft
---

Working with the release and deployment of the 

## Prerequisites

What if I wanted to create two release paths? I wanted one that only deploys to `development` where another that would skip development and go through `Test`, `UAT` to `Production`.

Lifecycles and Channels are the way to do it.

A Lifecycle is a high level flow flow between different phases of a project. 