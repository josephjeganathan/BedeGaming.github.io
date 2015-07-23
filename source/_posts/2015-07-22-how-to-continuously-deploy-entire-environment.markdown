---
layout: post
title: "How to continuously deploy an entire environment"
date: 2015-07-23 18:00:00 +0100
comments: true
author: Rytis Bieliunas
categories: agile automation cd octopusdeploy
---

At Bede we are using [Octopus Deploy](https://octopusdeploy.com/) as our deployment automation system. We use it to promote components of our platform through various environments, from Integration, through Development and QA, and finally to Production. It's worked wonderfully well for us so far. However Bede has grown. We have near 100 different projects in Octopus and now, more importantly, multiple Production environments.

<!-- more -->

Bede gaming platform is split into lots of small services which keeps development teams reasonably independent and productive as the platform grows, but it also makes deployment orchestration tricky. We want to make it easy to move changes through environments and so tools and processes are needed that minimize manual intervention and human error.

Here is the issue in a single picture:

{% img center /images/odfullview.png 'Octopus Deploy in Bede' 'Octopus Deploy in Bede' %}
*There are many components! They must work well together and Octopus Deploy at this point only has tooling to work with one component at a time.*

What we needed was a way to take a QA'd environment, consisting of the entire set of components, and deploy that set, as one, to another environment.  This is what we're calling a "Platform Version".

Lucky for us Octopus Deploy exposes a complete API. That means every thing that you can do as a user can be automated.

And so we are building: **Conan The Deployer**

{% img center /images/conan.jpg 'Conan The Deployer' 'Conan The Deployer' %}
*He looks innocent enough here, but he can mess up your entire production environment (Photo courtesy Universal Pictures/Everett Collection)*

Conan essentially acts as a coordination and orchestration layer on top of Octopus Deploy, and a source of truth for the state of any given OD environment. It records the state of an environment, either on demand or to schedule. We're also considering it could coordinate between multiple instances of Octopus Deploy, when for example you may have an OD instance per-client.

Quick rundown of how it works:

First thing you need to do is take a snapshot, which captures all component release versions on selected environment:

{% img center /images/conan_dashboard.png 'Conan Dashboard' 'Conan Dashboard' %}

Then you compare it to an environment where you want to deploy that snapshot. This lets you see which components the deployment would upgrade, downgrade or ignore.

{% img center /images/conan_compare.png 'Conan Compare' 'Conan Compare' %}

Logging in to Conan is done using normal Octopus Deploy user credentials and all Conan actions are initiated using those credentials. That means access control, user management and some audit come for free.

Review checks with Octopus if your user has access to deploy everything selected from the snapshot. Here you see final set of components and deploy the entire thing with one click of a button.

{% img center /images/conan_review.png 'Conan Review' 'Conan Review' %}

This is an early proof of concept project. We have plans for it!
