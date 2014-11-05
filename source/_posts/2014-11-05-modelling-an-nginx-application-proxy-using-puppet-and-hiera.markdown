---
layout: post
title: "Modelling an nginx Application Proxy using Puppet and Hiera"
date: 2014-11-05 15:53:08 +0000
comments: true
author: Rob Rankin
categories: Tech
keywords: "pupet, hiera, roles, profiles, nginx, proxy, iteration"
description: "Modelling an nginx Application Proxy using Puppet and Hiera"
---
Goal
====

To model nginx configurations in Hiera, extracting those configurations
from the Puppet DSL, and create a Puppet Profile to combine that with
the nginx module.

Links
=====

Puppet Docs
-----------

* [Puppet Resource Abstraction Layer](https://docs.puppetlabs.com/learning/ral.html)
* [Puppet create_resources](https://docs.puppetlabs.com/references/latest/function.html#createresources)
* [Scoping](https://docs.puppetlabs.com/guides/scope_and_puppet.html)
* [Namespacing](https://docs.puppetlabs.com/puppet/latest/reference/lang_namespaces.html)

Puppet Roles/Profiles
---------------------

* [Craig Dunn, Designing Puppet Talk](http://www.slideshare.net/PuppetLabs/roles-talk)
* [End to End Roles and Profiles Example](https://ask.puppetlabs.com/question/1655/an-end-to-end-roleprofile-example-using-hiera/)
* [Puppet Workflow Part 1](http://garylarizza.com/blog/2014/02/17/puppet-workflow-part-1/)
* [Puppet Workflow Part 2](http://garylarizza.com/blog/2014/02/17/puppet-workflow-part-2/)

Hiera
-----

* [Puppet Labs Hiera Docs](https://docs.puppetlabs.com/hiera/1/)
* [Passing Variables to Hiera](https://docs.puppetlabs.com/hiera/1/variables.html#passing-variables-to-hiera)

nginx Module
------------

* [jfryman nginx Puppet Module](https://github.com/jfryman/puppet-nginx)

nginx
-----

* [nginx Project Page](http://wiki.nginx.org/Main)

Requirements
============

Within nginx we need to define a set of nginx "server" contexts with the
same set of "location" contexts in each "server".

Think of this as "http://SiteA.com" and "http://SiteB.com", that both
have the same set of "locations", which define the nginx upstreams
(backends), that combined provide a single site that is created from
disparate SOA applications.

This is essentially a model to build a nginx based proxy layer sitting
in front of a multi-tenant SOA application stack.

<!-- more -->

Approach
--------

*Example 1: Single Server, Two Locations modelled in Hiera (YAML)*

```
proxy::sites::vhosts:
  SiteA:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false

proxy::sites::locations:
  root:
    vhost : SiteA
    location : '/'
    ensure : 'present'
    proxy : 'http://serviceA'
  server_status: 
    vhost : SiteA
    location : '/server-status'
    ensure : 'present'
    stub_status : true
```

In this example we have a single "server", called SiteA. SiteA also has
2 locations defined, "root", and "server-status".

The relationship between the server and location is defined by the
parameter "vhost" in the location stanza; by being set to "SiteA", the
location stanza is created within the SiteA server stanza, when all this
is realized into an actual nginx configuration file.

There are possibly other Hiera/YAML structures that would allow us to
model this relationship more clearly, however we're using this specific
structure because we want to take advantage of Puppets
`create_resources` function in conjunction with the nginx module:

-   <https://docs.puppetlabs.com/references/latest/function.html#createresources>

Essentially, with the data structured in the correct manner we can pass
it to `create_resources` as a Puppet hash and it will create the
specified resource. In this case the specific resources are the location
and vhost resources defined in the nginx module. By defining our config
data in Hiera in the right structure, we can use `create_resources` and
make our life a bit simpler.

This presents a challenge when you want to create more than one server
(multiple vhosts).

*Example 2: Two vhosts, Two locations*

```
proxy::sites::vhosts:
  SiteA:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false
  SiteB:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false

proxy::sites::locations:
  root:
    vhost : SiteA
    location : '/'
    ensure : 'present'
    proxy : 'http://serviceA
  server_status:
    vhost : SiteA
    location : '/server-status'
    ensure : 'present'
    stub_status : true
```

You now have 2 servers/vhosts. However the location stanzas are only
related to the first server/vhost (SiteA), as specified by the locations
vhost paramater.

In order to create a "root" and "server_status" location for SiteB,
following the same model as above, you would end up with:

*Example 3: Two vhosts, Four locations (2 per vhost)*

```
proxy::sites::vhosts:
  SiteA:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false
  SiteB:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false

proxy::sites::locations:
  SiteA_root:
    vhost : SiteA
    location : '/'
    ensure : 'present'
    proxy : 'http://serviceA
  SiteA_server_status:
    vhost : SiteA
    location : '/server-status'
    ensure : 'present'
    stub_status : true
  SiteB_root:
    vhost : SiteB
    location : '/'
    ensure : 'present'
    proxy : 'http://serviceA
  SiteB_server_status:
    vhost : SiteB
    location : '/server-status'
    ensure : 'present'
    stub_status : true
```

This is obviously suboptimal as you're repeating configuration for both
locations now. For each vhost SiteA and SiteB, you need to define 2
locations.

You'll also notice that the name of the locations have now changed to
"SiteA_root", "SiteA_server_status" and "SiteB_root",
"SiteB_server_status". This is due to Puppet not allowing duplicate
resource declarations, which these locations would be if they were both
named "root" and "server_status". So we create unique resources, by
making the resource names unique.

You can see where this is leading. For every vhost that we want to add,
that has the same set of locations, we'll end up duplicating the
location data, and just changing the location name and the vhost the
location is associated with. This is probably bad, at least it feels
very bad to me.

The natural response to this is "Simples! I'll just iterate over the
list of vhosts and create the set of locations for each server!".

Nope. Not really.

Puppet DSL doesn't really do iteration. Not naturally anyhow.

My favorite comment on this:

-   <http://stackoverflow.com/questions/12958114/how-to-iterate-over-an-array-in-puppet/13008766#13008766>

Quote: "The Puppet developers have irrational prejudices against
iteration based on a misunderstanding about how declarative languages
work."

(Disclaimer: I'm not a dev, I won't attempt to evaluate the truth of
that statement, but I still think its amusing)

So we have 2 problems:

1.  Iteration in Puppet is not… normal, allowed, right, easy?
2.  The "names" of the locations need to be unique, which means they
    can't simply be text labels in YAML. There must be some
    interpolation somewhere.

We'll deal with the second problem first, as it's a slightly easier
problem to solve, at least in my experience.

Hiera Interpolation
-------------------

Hiera/Puppet allows variables to be passed into Hiera:

-   <https://docs.puppetlabs.com/hiera/1/variables.html#passing-variables-to-hiera>

Quote: "When used with Puppet, Hiera automatically receives all of
Puppet's current variables. This includes facts and built-in variables,
as well as local variables from the current scope".

This means we can pass Hiera some variable defined within the Puppet
Manifest, and hopefully therereby creating dynamically named nginx
location resources within Puppet. Probably the most important bit of
that quote is that Hiera has access to "local variables from the current
scope". That'll be touched upon further on in this article.

So, that would look something like Example 4.

*Example 4: Two vhosts, 2 dynamically named locations*

```
proxy::sites::vhosts:
  SiteA:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false
  SiteB:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false

proxy::sites::locations:
  "%{vhost}_root":
    vhost : "%{vhost}"
    location : '/'
    ensure : 'present'
    proxy : 'http://serviceA
  "%{vhost}_server_status":
    vhost : "%{vhost}"
    location : '/server-status'
    ensure : 'present'
    stub_status : true
```

So, without getting into the intricate details, this suggests that the
nginx location resources will be named `<vhost>_<location>`. The goal
would be to end up with 4 locations, 2 assigned to each vhost.

*Example 5: Locations per Vhost*

<table style="width:100%">
  <tr>
    <th>Vhost</td>
    <th>Location Name</td> 
  </tr>
  <tr>
    <td>SiteA</td>
    <td>SiteA_root</td> 
  </tr>
  <tr>
    <td></td>
    <td>SiteA_server_status</td> 
  </tr>
  <tr>
    <td>SiteB</td>
    <td>SiteB_root</td> 
  </tr>
  <tr>
    <td></td>
    <td>SiteB_server_status	</td> 
  </tr>
</table>

The next problem to solve is how to iterate over the list of vhosts.

Puppet Iteration
----------------

As mentioned, Puppet doesn't really do iteration natively (ignoring the
future parser for now). So how do we accomplish this using native Puppet
DSL?

The first thing to realize is that iteration does in fact happen (or
something similar enough that the difference doesnt matter to most
people). A very good and clear blog article about this can be found
here:

-   <https://tobrunet.ch/2013/01/iterate-over-datastructures-in-puppet-manifests/>

The basic point being made is that native Puppet functions will iterate
over a Puppet array, executing for each array member.

The first example on that blog post is as follows.

*Example 6: Iteration in Puppet DSL*

```
define arrayDebug {
 notify { [Item ${name}]() }
}

class array {
 $array = [ 'item1', 'item2', 'item3' ]
 arrayDebug { $array: }
}
```

Essentially the `arrayDebug` resource (define) will iterate over the
array members of the array `$array`.

Creating a Puppet Profile
-------------------------

So the next step is to write a Puppet "Profile" for nginx that
encapsulates the functionality we want, using Hiera as the data source,
and the nginx module as the underlying component.

Lets start by creating the basic Hiera config needed for a very simple
nginx install.

*Example 7: Hiera for nginx*

```
nginx::default::mail : false
nginx::default::worker_processes : %{processorcount}
nginx::default::server_tokens : 'off'
nginx::default::nginx_error_log : '/var/log/nginx/error.log debug'
nginx::default::http_access_log : '/var/log/nginx/access.log'
nginx::default::proxy_cache_path : '/var/cache/nginx'
nginx::default::proxy_cache_levels : '2'
nginx::default::proxy_cache_keys_zone : 'cache:10m'
nginx::default::proxy_cache_max_size : '2048m'
```

There's nothing exceptional here, this is all nginx boilerplate to set
some global defaults for nginx. The interesting bit of this is when we
start creating the nginx Profile.

*Example 8: Base nginx Profile in Puppet DSL*

 
```
class profile::linux::nginx {

  $mail							= hiera('nginx::default::mail')
  $worker_processes				= hiera('nginx::default::worker_processes')
  $server_tokens				= hiera('nginx::default::server_tokens')
  $nginx_error_log				= hiera('nginx::default::nginx_error_log')
  $http_access_log				= hiera('nginx::default::http_access_log')
  $proxy_cache_path				= hiera('nginx::default::proxy_cache_path')
  $proxy_cache_levels			= hiera('nginx::default::proxy_cache_levels')
  $proxy_cache_keys_zone		= hiera('nginx::default::proxy_cache_keys_zone')
  $proxy_cache_max_size			= hiera('nginx::default::proxy_cache_max_size')
  $names_hash_bucket_size		= hiera('nginx::default::names_hash_bucket_size')

  #################################################
  # Create the base nginx config by passing the nginx module the minimum config that we use across all our systems
  #################################################

  class { '::nginx':
    mail						=> $mail,
    worker_processes			=> $worker_processes,
    server_tokens 				=> $server_tokens,
    nginx_error_log 			=> $nginx_error_log,
    http_access_log 			=> $http_access_log,
    proxy_cache_path 			=> $proxy_cache_path,
    proxy_cache_levels 			=> $proxy_cache_levels,
    proxy_cache_keys_zone 		=> $proxy_cache_keys_zone,
    proxy_cache_max_size 		=> $proxy_cache_max_size,
    names_hash_bucket_size 		=> $names_hash_bucket_size,
  }
}
```

So this is all pretty simple again. This is a Puppet class called
`profile::linux::nginx`, which sets some default variables, and passes
them to another class called `::nginx`.

This class is a "Profile" simply by dint of it being created within the
Profile module / namespace. It's technically no different than any other
Puppet module, it's simply a widely accepted convention that Puppet
Roles and Profiles are modules.

The `::nginx` class actually refers to the nginx module, denoted by the
leading `::`, basically meaning "at top scope, named nginx".

If you're wondering about the `::` colon syntax, start by reading about
Puppet scope and namespacing:

-   <https://docs.puppetlabs.com/guides/scope_and_puppet.html>
-   <https://docs.puppetlabs.com/puppet/latest/reference/lang_namespaces.html>

So this nginx Profile achieves the very first of our goals; that is to
install nginx and set some very basic global nginx parameters.

The next step would be create the nginx "servers" (called vhosts here).
In order to achieve this step we need to retrieve the vhost
configuration data from Hiera. We'll then use that data to create the
vhosts.

*Example 9: nginx Profile with vhosts being created*

```
class profile::linux::nginx {

  $mail							= hiera('nginx::default::mail')
  $worker_processes				= hiera('nginx::default::worker_processes')
  $server_tokens				= hiera('nginx::default::server_tokens')
  $nginx_error_log				= hiera('nginx::default::nginx_error_log')
  $http_access_log				= hiera('nginx::default::http_access_log')
  $proxy_cache_path				= hiera('nginx::default::proxy_cache_path')
  $proxy_cache_levels			= hiera('nginx::default::proxy_cache_levels')
  $proxy_cache_keys_zone		= hiera('nginx::default::proxy_cache_keys_zone')
  $proxy_cache_max_size			= hiera('nginx::default::proxy_cache_max_size')
  $names_hash_bucket_size		= hiera('nginx::default::names_hash_bucket_size')

  #################################################
  # Get the hash of vhosts from Hiera. Extract the hash keys into a list (array).
  #################################################

  $vhosts						= hiera('proxy::sites::vhosts')

  #################################################
  # Create the base nginx config by passing the nginx module the minimum config that we use across all our systems
  #################################################

  class { '::nginx':
    mail						=> $mail,
    worker_processes			=> $worker_processes,
    server_tokens 				=> $server_tokens,
    nginx_error_log 			=> $nginx_error_log,
    http_access_log 			=> $http_access_log,
    proxy_cache_path 			=> $proxy_cache_path,
    proxy_cache_levels 			=> $proxy_cache_levels,
    proxy_cache_keys_zone 		=> $proxy_cache_keys_zone,
    proxy_cache_max_size 		=> $proxy_cache_max_size,
    names_hash_bucket_size 		=> $names_hash_bucket_size,
  }

  ################################################
  # Create the ngingx vhosts (server blocks in nginx config terms). Pass create_resources the hash retrieved from Hiera,
  # into the nginx::resource::vhost resource defined by the nginx module
  ################################################

  create_resources('::nginx::resource::vhost', $vhosts)

}
```

First we add some functionality to retrieve the vhost config from Hiera
using a Hiera lookup function in Puppet:

    $vhosts = hiera('proxy::walletv2::vhosts')

We then pass the resultant hash to a Puppet function called
`create_resrouces`:

    create_resources('::nginx::resource::vhost', $vhosts)

`create_resources` is a very useful Puppet function. Given the right set
of information, in the right structure, it will auto-magically create
the specified resource. In this case the resource we're talking about is
the `::nginx::resource::vhost` resource. This resource is defined in the
nginx module, as denoted by the top scope `::nginx`. The hash `$vhosts`
contains the data retrieved from Hiera, specifically from the key
`proxy::sites::vhosts` found in Example 2 to Example 4.

This will create the nginx vhost (server) configurations in the actual
nginx config files, using the key/values as parameters.

Further reading:

-   <https://docs.puppetlabs.com/references/latest/function.html#createresources>
-   <https://docs.puppetlabs.com/learning/ral.html>

The final step is to combine our interpolated Hiera data and Puppet
iteration learnings into the Profile so that each vhost that gets
created, also gets the same set of locations.

*Example 10: nginx Profile with vhosts and locations being created*

```
class profile::linux::nginx {

  $mail							= hiera('nginx::default::mail')
  $worker_processes				= hiera('nginx::default::worker_processes')
  $server_tokens				= hiera('nginx::default::server_tokens')
  $nginx_error_log				= hiera('nginx::default::nginx_error_log')
  $http_access_log				= hiera('nginx::default::http_access_log')
  $proxy_cache_path				= hiera('nginx::default::proxy_cache_path')
  $proxy_cache_levels			= hiera('nginx::default::proxy_cache_levels')
  $proxy_cache_keys_zone		= hiera('nginx::default::proxy_cache_keys_zone')
  $proxy_cache_max_size			= hiera('nginx::default::proxy_cache_max_size')
  $names_hash_bucket_size		= hiera('nginx::default::names_hash_bucket_size')

  #################################################
  # Get the hash of vhosts from Hiera. Extract the hash keys into a list (array).
  #################################################

  $vhosts						= hiera('proxy::sites::vhosts')
  $vhostslist					= keys($vhosts)

  #################################################
  # Create the base nginx config by passing the nginx module the minimum config that we use across all our systems
  #################################################

  class { '::nginx':
    mail						=> $mail,
    worker_processes			=> $worker_processes,
    server_tokens 				=> $server_tokens,
    nginx_error_log 			=> $nginx_error_log,
    http_access_log 			=> $http_access_log,
    proxy_cache_path 			=> $proxy_cache_path,
    proxy_cache_levels 			=> $proxy_cache_levels,
    proxy_cache_keys_zone 		=> $proxy_cache_keys_zone,
    proxy_cache_max_size 		=> $proxy_cache_max_size,
    names_hash_bucket_size 		=> $names_hash_bucket_size,
  }

  ################################################
  # Create the ngingx vhosts (server blocks in nginx config terms). Pass create_resources the hash retrieved from Hiera,
  # into the nginx::resource::vhost resource defined by the nginx module
  ################################################

  create_resources('::nginx::resource::vhost', $vhosts)

  define profile::linux::nginx::location {
    ### Creating Locations
    $locations = hiera('proxy::sites::locations')
    $vhostslocations = { vhost => $name }

    create_resources('::nginx::resource::location', $locations, $vhostslocations)
  }

  ################################################
  #
  # This calls the above define per member of $vhostslist array
  #
  ################################################

  profile::linux::nginx::location { $vhostslist: }

}
```

So what have we done here?

The first thing is that we've added an array called `$vhostslist`, whos
array members are made up of the key names of the `$vhosts` hash:

    $vhostslist = keys($vhosts)

This gives us the array that we want to iterate over, as discussed in
the Puppet iteration section. The array members in this example will be
"SiteA" and "SiteB", i.e. `[ "SiteA", "SiteB" ]`.

The second thing we've added is a new define,
`profile::linux::nginx::location`:

```
define profile::linux::nginx::location {
  ### Creating Locations
  $locations = hiera('proxy::sites::locations')
  $vhostslocations = { vhost => $name }

  create_resources('::nginx::resource::location', $locations, $vhostslocations)
 }
```

This can be thought of as a function in more general programming
languages, that will iterate over the array `$vhostslist`. Within this
define we are retrieving the list of locations into a has called
`$locations`, using another Hiera lookup, on the key
`proxy::sites::locations` (Yes I'm aware this will be done for every
iteration, not optimal). We then create a new hash called
`$vhostslocations` which contains the key `vhost` and value `$name`,
i.e. `{ vhost => $name }`.

Once we have the `$locations` hash and the `$vhostslocations` hash, we
then call `create_resources`. Just like the previous time we called
`create_resources`, the specific resource is defined by the nginx
module, and is called `::nginx::resource::location`. However, in this
case we're giving the `create_resources` function two items, not one.
The first item is the list of locations we'd like to create, and the
second is the `$vhostslocations` hash.

The second optional item passed to `create_resources` (
`$vhostslocations` ) is an anonymous hash of extra parameters to add to
the resource being created.

This define is then "called" by the line:

    profile::linux::nginx::location { $vhostslist: }

Before we go further, we need to look at the Hiera data again, and
modify it slightly:

*Example 11: Modified Hiera data*

```
proxy::sites::vhosts:
  SiteA:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false
  SiteB:
    listen_port : 8080
    rewrite_to_https : false
    use_default_location : false

proxy::sites::locations:
  "%{name}_root"
    location : '/'
    ensure : 'present'
    proxy : 'http://serviceA'
  "%{name}_server_status"
    location : '/server-status'
    ensure : 'present'
    stub_status : true
```

The change here is the variable we want interpolated has been changed to
`%{name}`, and the `vhost` parameter has been removed.

You'll notice in Example 10 the newly added define suddenly makes use of
a variable called `$name`. We're also now referencing it in our modified
Hiera config in Example 11.

So what is `$name`? Well, this goes back to Puppet resources again. The
define `profile::linux::nginx::location` **is** a Puppet resource, and
as such must be "named" (and again, named uniquely). When we call the
define:

    profile::linux::nginx::location { $vhostslist: }

We're passing it the array that it iterates over, but under the hood in
a bit of Puppet magic, the "name" of the resource becomes the value of
the array member. So there will be two resources of type
`profile::linux::nginx::location`, one named "SiteA" and the other named
"SiteB". Within the scope of each of these resources, Puppet creates and
makes available a variable called `$name`, which of course is set to the
"name" of the resource, or in our case "SiteA" and SiteB".

Since we are performing a Hiera lookup within the resource (define)
`profile::linux::nginx::location`, that Hiera lookup now has access to
all the Puppet variables within that scope (see previous info about
Hiera being passed Puppet variables). Therefore the `%{name}` Hiera
variable is set to the `$name` Puppet variable. Magic.

The final piece of the puzzle is the `$vhostslocations` hash, and the
actual use of the `$name` variable. According to the `create_resources`
documentation:

"The values given on the third argument are added to the parameters of
each resource present in the set given on the second argument."

So therefore the hash `$vhostslocations { vhost => "$name" }` basically
gives the nginx module an extra parameter that is applied to each
location created of "vhost => SiteA" or "vhost => SiteB". This
establishes the relationship between the locations and their parent
vhosts.

Put it all together and we now have a start towards cleanly modelling
nginx configurations in Hiera, without repeating data, and using the
Roles and Profile pattern. This allows us to completely extract site
configuration data from the Puppet DSL, by providing an abstraction
layer to the nginx module (the nginx Profile). I didn't actually touch
upon a Puppet Role here, but once you have your Profile(s) created, the
creation of Roles is generally much simpler, as they should be nothing
more than a collection of Profiles. The links provided at the beginning
of this post on Roles/Profiles should make the rest clear.

So that's basically an end-to-end example (although not necessarily 100%
working) of using Hiera to model nginx, and combining that with an
abstraction layer (the Profile) above the nginx module.

Possible re-factors
-------------------

1.  Duplicate data in the vhosts configurations could be reduced using
    the same iteration trick used for the location configs.
2.  Removing the Hiera lookup from the `profile::linux::nginx::location`
    resource would prevent repeating the lookup.
3.  Remove the use of the `$vhostslocations` hash. Use Hiera
    interpolation to specify the vhost parameter in the location
    resource.
4.  Many others I haven't thought of :)

If anyone has any improvements, suggestions, comments or criticisms let
me know at mahhy at undertow dot ca.
