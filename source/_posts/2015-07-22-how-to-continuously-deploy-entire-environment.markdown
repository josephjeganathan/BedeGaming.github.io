---
layout: post
title: "How to continuously deploy an entire environment"
date: 2015-07-22 15:00:00 +0100
comments: true
author: Rytis Bieliunas
categories: agile automation cd octopusdeploy
---

At Bede we are using Octopus Deploy as our deployment automation system. We use it to promote the numerous components of our platform through various environments, from Integration, through Development and QA, and finally to Production. It's worked wonderfully well for us so far. However Bede has grown. We have near 100 different projects in Octopus and now, more importantly, multiple Production environments.

What we needed was a way to take a QA'd environment, consisting of the entire set of components, and deploy that set, as one, to another environment.  This is what we're calling a "Platform Version".
<!-- more -->

Lucky for us Octopus Deploy exposes a complete API. That means every thing that you can do as a user can be automated.

And so we are building: **Conan The Deployer**

{% img center /images/conan.jpg 'Conan The Deployer' 'Conan The Deployer' %}
*He looks innocent enough here, but he can mess up your entire production environment*
*Photo courtesy Universal Pictures/Everett Collection*

Conan essentially acts as a coordination and orchaestration layer on top of Octopus Deploy, and a source of truth for the state of any given OD environment.  Conan records the state of an environment, either on demand or to schedule.  Conan can coordinate between multiple instances of Octopus Deploy, when for example you may have an OD instance per-client.

Quick rundown of what we're using it for:

First thing you need to do is take a snapshot, which captures all component release versions on selected environment:

{% img center /images/conan_dashboard.png 'Conan Dashboard' 'Conan Dashboard' %}

Then you compare it to an environment where you want to deploy that snapshot. This lets you see which components the deployment would upgrade, downgrade or ignore.

{% img center /images/conan_compare.png 'Conan Compare' 'Conan Compare' %}

Logging in to Conan is done using normal Octopus Deploy user credentials and we initiate all Octopus actions using those credentials. That means we get user management and access control for free.
Review checks with Octopus if your user has access to deploy everything selected from the snapshot. Here you see final set of components and deploy the entire thing with one click of a button.

{% img center /images/conan_review.png 'Conan Review' 'Conan Review' %}

This is an early proof of concept project. We have plans for it!
