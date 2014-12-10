---
layout: post
title: "Bede Gaming Puppet Module for Redis"
date: 2014-11-24 16:30:00 +0000
comments: true
author: Rob Rankin
categories: Tech
keywords: "pupet, modules, redis"
description: "BedeGaming Puppet Module for Redis"
---
As of last week we've created and released a Puppet Module for Redis.  While there are other Redis modules available on GitHub and the Forge, we think a lot of them are either overly complex or outdated in their design and methods.  Many of them also install from source packages, which we don't like.
<!-- more -->
We think our module is a bit better:

* Simple, clean layout
* Offers single entry point (the '::redis' class)
* Provides a full set of sane defaults
* Provides for all configuration options found in the redis.conf file
* Does all the basic installation, configuration and management you'd expect

One of the other major reasons for our own module was our desire to not use Redis that came from source packages.  We use CentOS for the Linux portion of our platform, and are pretty religious about only installing from RPM's.  This module expects that there will be a Redis RPM package available somewhere within the yum repos you've made available on your systems, which we feel is good design, as it leaves it up to you to implement that package source.  That's not the job of a Redis module.

Currently the Redis module is not on the Forge, but probably will be in the near future.