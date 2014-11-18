---
layout: post
title: "BedeGaming Puppet Module"
date: 2014-11-18 16:30:00 +0000
comments: true
author: Rob Rankin
categories: Tech
keywords: "pupet, modules, foundationdb, redis"
description: "BedeGaming Puppet Modules"
---
In the past month or so Bede Gaming has released 2 Puppet Modules, one for FoundationDB and one for Redis.

<!-- more -->

[FoundationDB Module](https://github.com/BedeGaming/puppet-foundationdb)
---
Here at Bede we've been using FoundationDB for some time, and are basically in love with it.  Fully distributed, ACID compliant transactions?  Yes please.  It's fast, it scales, it's reliable and redundant.  From an operations persons point of view it's simply sexy.

However, it is a relatively new technology, with a small but growing community.  That means that the ecosystem surrounding FDB is not quite as extensive as many of the other, longer lived persistence technologies.  Therefore there was no Puppet modules available to manage installation and configuration, so we had to write our own.

The [module](https://github.com/BedeGaming/puppet-foundationdb) can manage installation, configuration, upgrading, etc.  All the usual bits'n'bobs you'd expect a Puppet module to manage.  This does come with one caveat however:  it doesn't currently manage FDB cluster membership, due to the way FDB itself manages membership.  We're still exploring ways to add at least some amount of membership management into the module, and of course if anyone wants to contribute towards the module, feel free to fork and issue PR's.

The FoundationDB module can be found on the [Forge](https://forge.puppetlabs.com/bedegaming/foundationdb).

[Redis Module](https://github.com/BedeGaming/puppet-redis)
---
Just today we've created and released a Redis module as well.  While there are other Redis modules available on GitHub and the Forge, we think a lot of them are either overly complex or outdated in their design and methods.  Many of them also install from source packages, which we don't like.

We think our module is a bit better:

* Simple, clean layout
* Offers single entry point (the '::redis' class)
* Provides a full set of sane defaults
* Provides for all configuration options found in the redis.conf file
* Does all the basic installation, configuration and management you'd expect

One of the other major reasons for our own module was our desire to not use Redis that came from source packages.  We use CentOS for the Linux portion of our platform, and are pretty religious about only installing from RPM's.  This module expects that there will be a Redis RPM package available somewhere within the yum repos you've made available on your systems, which we feel is good design, as it leaves it up to you to implement that package source.  That's not the modules job.

Currently the Redis module is not on the Forge, but probably will be in the near future.