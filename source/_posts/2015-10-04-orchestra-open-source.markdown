---
layout: post
title: "Orchestra - Bede Gaming's Open Source JavaScript Framework"
date: 2015-10-04 15:00:00 +0100
comments: true
author: Andrew Humphreys
categories: javascript
---

My previous posts on Orchestra have detailed the work that Bede employees have put in to move the framework from an application scaffold to a library useful for a multitude of applications. This work finally paid off when Bede released Orchestra to the JavaScript community.

<!-- more -->

Before the cynics jump onboard with the "not another JavaScript framework" statements, let me explain the reasoning behind the release and the different approach we are taking in comparsion to the competing MV* frameworks. Orchestra is not aiming to become an MV* framework, it is aiming to extend an existing, popular framework to provide a better entry point for adoption.

Being a "core team" member for Backbone.Marionette, I spend most of my free time contributing to the popular Open Source project, Orchestra being released will not change that. In fact, it is my standing in the Marionette community that has pushed me to persue this release. Some of the common questions the team are asked are _"How do I start building my Marionette application?"_ or _"Where can I find a set of best practices for Marionette applications?"_. Additionally, dependency problems are  common issues we find opened against our repo. The problem with maintaining such an unopinionated library is that there are so many examples of application architecture out there: some are brilliant, some are terrible. This is what Orchestra is looking to solve.

Orchestra contains a collection of popular libraries used in many Marionette applications and aims to say "If you are building a Marionette application, these are the tools you need". Dependency problems will be gone as Orchestra pulls in a known set of compatible versions, the library follows SemVer so you can trust we will not break your applications.

Our library is more than just a collection of other libraries though, we have included helper methods used in production by Bede on our multi lingual, multi platform applications. Need to format a number as a currency - use our currency helper, need to change that currency from GBP to Euros for a particular theme, pass the helper the Euro locale and it will change it for you. If you are developing an application for use in multiple countries, our translation helper will allow you to use the power of i18n to provide locale specifc strings for each text item you display.

For developers looking to launch applications on touch devices, Orchestra's touch helper extends Backbone's View with `hammerEvents` allowing Views to listen for touch actions. There are more helpers such as Visibility and Local Storage, be sure to read our [documentation](https://github.com/BedeGaming/orchestra/blob/master/README.md) to find out more.

The Orchestra library is still evolving and will continue to grow alongside Marionette, Marionette 3.0 will bring some great new features and, inspired by [Marionette Wires](https://github.com/thejameskyle/marionette-wires), we are continuing development on an official Orchestra Yeoman generator. It is my hope that this will give those who search for opinions on Marionette application development the starting point they need to rapidly produce top quality, well architected JavaScript applications.

To keep up to date on Orchestra development, star our [repo](https://github.com/BedeGaming/orchestra/), or even better "fork" it and start contributing!

