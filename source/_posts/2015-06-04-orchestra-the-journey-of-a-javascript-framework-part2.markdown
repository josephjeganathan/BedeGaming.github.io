---
layout: post
title: "Orchestra - The Journey of a Javascript Framework Part 2"
date: 2015-06-04 15:00:00 +0100
comments: true
author: Andrew Humphreys
categories: javascript
---

In part 1 of of this series I wrote about how we refined our build process using Gulp and Browserify, these changes also made a significant impact on the structure of the Orchestra Framework. Part 2 covers how Orchestra transitioned from an all encompassing framework to a library that brings together our favourite tools.

<!-- more -->

The initial vision for Orchestra was to create an application boilerplate that made use of a common toolset we could use for games. Libraries such as [MarionetteJS](http://marionettejs.com/), [Phaser.io](http://phaser.io/) and [HowlerJS](https://github.com/goldfire/howler.js/) are critical for us to deliver rich gaming content, across multiple platforms. The initial application scaffold looked something like this:

{% img center /images/initial-app-structure.png 'initial application folder structure' %}

Drilling further down the tree into the `components` directory you can see our 'reuseable' components:

{% img center /images/components-directory.png 'initial components directory' %}

The image aboves shows the components for one of our games, these components are meant to be reuseable so, how do we include `orchestra-chat` in another repo? We copy/paste it. This obviously leads to a lot of code duplication across different applications. We had 2 applications running this way before we grew tired of ensuring both versions were up to date. With more applications in the pipeline we decided to restructure our boilerplate.

## Browserify to the rescue

Browserify makes use of node's CommonJS module loader, to load our component we simply do:

```
var component = require('app/components/orchestra-chat');
```

However, by default the loader will first search the `node_modules` folder (by default all NPM packages are installed here), with this in mind we converted our components into NPM packages so they could be installed via NPM. Now to load our component we do:

```
var component = require('orchestra-chat');
```

Utilising the NPM package manager has enabled us to remove components from the applications and include them in the applications dependencies so they are installed at build time. This brings our folder structure to what we have today:

{% img center /images/new-folder-structure.png 'new application folder structure' %}

In addition to the streamlined folder structure, we are now able to avoid duplicating code across multiple repositories, utilising the power of the NPM package manager. You may also have noticed in the image above, we have removed `helpers`, `mixins` and `models`. These directories also contained common code shared between applications, for example, a translation helper, a currency display helper or a mixin for touch events. This was code that didn't belong in any particular component but was more part of the framework itself. What we did next was create an Orchestra library which became a dependency of the applications.

## Orchestra the library

We needed to remove this common code from applications, the next logical step was to convert Orchestra itself into a library. It has proven to be an inspired decision. Orchestra is now a library with it's own opinions about what tools to use for certain jobs, it extends Marionette and [Backbone.Radio](https://github.com/marionettejs/backbone.radio), provides access to utility libraries such as [lodash](https://lodash.com/) and helpers for things such as translation, currency and local storage.

This reduces code duplication and simplifes our modules, rather than pulling in each dependency individually, we simply include Orchestra and know that it has all the tools we need.

## The Future...

The aim for Orchestra would be to release it to the community and Open Source the library. Tthere is still some work to do - but it isn't far off. 
Our [Yeoman generator]( http://yeoman.io/) is quite out of date and needs updating for our new application structure, PR's welcome [here](https://github.com/BedeGaming/protege). 
Now that Orchestra is a library rather than a framework, we aren't necessarily limited to game applications.

**Orchestra: Client-side ~~Game~~ CMS Widget Orchestration ~~Framework~~ Library**

Coming next in this series will be a post on how our applications hang together, an app having lots of separate components needs a good method of communcation. It will go in depth with our event architecture, and explain why [Facebook's Flux Architecture](https://facebook.github.io/flux/) maybe isn't quite as innovative as many think...





