---
layout: post
title: "BedeGaming Puppet Module for FoundationDB"
date: 2014-11-18 16:30:00 +0000
comments: true
author: Rob Rankin
categories: Tech
keywords: "pupet, modules, foundationdb"
description: "BedeGaming Puppet Module for FoudnationDB"
---
Here at Bede we've been using [FoundationDB](https://foundationdb.com/) for some time, and are basically in love with it.  Fully distributed, ACID compliant transactions?  Yes please.  It's fast, it scales, it's reliable and redundant.  From an operations persons point of view it's simply sexy.

However, it is a relatively new technology, with a small but growing community.  That means that the ecosystem surrounding FDB is not quite as extensive as many of the other, longer lived persistence technologies.  Therefore there was no Puppet modules available to manage installation and configuration, so we had to write our own.

The [module](https://github.com/BedeGaming/puppet-foundationdb) can manage installation, configuration, upgrading, etc.  All the usual bits'n'bobs you'd expect a Puppet module to manage.  This does come with one caveat however:  it doesn't currently manage FDB cluster membership, due to the way FDB itself manages membership.  We're still exploring ways to add at least some amount of membership management into the module, and of course if anyone wants to contribute towards the module, feel free to fork and issue PR's.

The FoundationDB module can be found on the [Forge](https://forge.puppetlabs.com/bedegaming/foundationdb).