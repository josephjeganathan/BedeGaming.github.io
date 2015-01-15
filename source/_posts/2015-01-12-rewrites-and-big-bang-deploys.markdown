---
layout: post
title: "Rewrites and big bang deploys"
date: 2015-01-12 12:37:14 +0000
comments: true
categories: deployment microservices
author: Andy Wardle
---

Developers love to rewrite code:
* It's always more exciting to work on Greenfield development. 
* The code you are working on was written by someone else and it is poorly written, hard to understand, or worse * case, it doesn’t do what the business requires. 
* There are always shiny new things which developers want to use (reactive extensions, event sourcing etc. etc.) and it's not usually feasible to implement these in an existing service 
(note: I am not advocating rewriting a service just for sake of some new technology or style).

<!-- more -->

Most of the time, the business will not let developers spend months rewriting a service on a whim. Why spend a small fortune in developer time refactoring, with no tangible business value added?

Here at Bede we’ve been lucky (or possibly unlucky depending on your point of view) to have been given the chance to rewrite a few of our services from scratch. We’ve been able to show enough business benefit from doing it that the powers that be are willing to go a few months without any major feature development as they could see that short term pain will give long term gain.

Some of the reasons why we have gone for full rewrites over trying to just update the existing codebase:
* The existing service was written by an outsourced company, and wasn’t well managed, resulting in the service not meeting the business requirements
* The existing service was a monolith. Pulling functionality out and making single responsibility services gives the ability to do a rewrite (especially as we have a much better understanding of business requirements)
* The existing service is not scalable enough for future needs

Now that we have a bit of experience in performing these rewrites, we have honed the method of how we integrate them into the platform without causing any detrimental affect to users. The original service rewrites would be deployed as a ‘big bang’ release. It would immediately be released to all sites and players, and as good as our planning and testing might have been, when releasing code into the wild users always do things you don’t expect. Worse case is you aren’t even able to rollback the change and you perform firefighting to fix the issues as soon as possible.

We have two ways we can combat this to minimise impact to our players:
* Feature Switching
* Side by side running
 
###Feature Switching
Feature switching is a widely known topic and a lot of information about it is available across the internet. Within this blog post, I want to cover side by side running in a bit more depth, as I feel this has given us huge confidence in knowing how our new services will react when released into Production.

###Side By Side Running
{% img center /images/tee.png 'Request flow using Tee' 'Request flow using Tee' %}

The diagram above shows the flow of traffic. The request Tee is a very small application sitting beside NGINX that takes the incoming request and duplicates it. The main request carries on to the existing service and the response that is returned is sent back upstream as per normal. The duplicated request is then asynchronously sent to the new version of the service, the response back from the new service is logged and then sent to /dev/null. This allows us to test the new service, with real production traffic, without impacting the existing service.

Sitting in front of the new service is a micro service (Mapper). Its job is to take the incoming request, map it to the contract of the new service before sending it to the new service. It then does the same for the response, mapping from new contract to old contract. Having this mapper service in place allows us to switch to the new service without having to update all upstream clients in one go. We can update them as and when the resource is available. Once all upstream clients are updated to communicate directly with the new service, we can delete the mapper service.

###Data Comparison
The next benefit from doing side by side running is we can do data comparison between both services to make sure that data between services match. As the services within our platform all publish messages asynchronously, we created an application which consumes messages from both versions of the service and then compares the data. Any discrepancies are logged, which allows us to then find and resolve the issue.

We have used this method for our most recent service and it has given us the confidence to know that: 
* The new service runs as expected and the data is valid 
* The new service performs as expected (response times, memory / cpu usage etc) 
* We can release the new service with no impact to upstream clients


Going forward, if we ever need to do another service rewrite, we will be using both feature switching and side by side running to give us a high degree of confidence that we won’t negatively impact the gaming experience for our players.
