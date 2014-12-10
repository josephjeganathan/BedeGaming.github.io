---
layout: post
title: "Bede Gaming at µCon 2014"
date: 2014-12-10 14:15:55 +0000
author: Saverio Castelli
comments: true
categories: microservices, µCon, architecture
---

Microservices have gained more and more popularity in the software community during the past couple of years, up to the point that SkillsMatter has dedicated them an entire convention in London, µCon. I was lucky enough to participate  and here are some of the ideas that have been discussed.<!-- more -->

## What is a micro service?

It’s always a good idea to start with a clear definition, so here is what [Wikipedia](http://en.wikipedia.org/wiki/Microservices) has to say on the matter:

> "Microservices is a software architecture design pattern, in which complex applications are composed of small, independent processes communicating with each other using language-agnostic APIs. These services are small, highly decoupled and focus on doing a small task."

This seems appropriate, but leaves some open questions: how small a microservices should actually be? and what about the tasks? Somebody answers this questions by saying that a microservice shouldn’t be more than one hundred lines of code. Unfortunately, this looks like quite a silly rule of thumb. The best way to answer both questions is by thinking of microservices as Object Orientation applied to software design. A microservice becomes the counterpart of a class, and the single responsibility principle defines what small means: enough logic to be able to perform a single task, and no more than that. 

## What’s all the fuss about?

In the late ‘60, Conway made an interesting sociological observation:

> "organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations" [1]

This allows us identify a relation between Agile principles and Microservices: the principles have changed how software is built and how the teams interact, and, following from Conway’s law, this has changed how systems are designed. In some way, we could even go to the extent of saying that microservices are the natural expression of an Agile software development. 

As much as the sociological explanation can be interesting, microservices have also several technical  benefits:

1. Scalability : the only way you can scale a monolith is running several copies of the entire service. With a microservices approach, we can chose the which ones we need multiple instances of.
2. Easy to understand and maintain : being small and covering a single task, a microservice is much easier to understand and to maintain than a single, huge piece of software.
3. Easy to deploy : to bring even the smallest change to a monolith service in production, means deploy the entirety of it, with possible downtime and all the related problems. With microservices instead, the deployment of a single one of them can be done without bringing the entire infrastructure down.
4. Failure resilient : a failure in a monolithic application, means a failure of the entire service. On the other hand, the failure of a single microservice doesn’t reflect into a paralysis of the entire infrastructure.
5. Loosely coupled

## What’s the catch?

Unfortunately microservices are not the panacea of all evils. They indeed help removing the complexity from the service layer, but they push it down to the network layer. The two main problems that arise when dealing with a microservices infrastructure are discovery and handling of failures.

### Discovery

With a monolith, everything sits in one place, so resources location doesn’t really pose any issue. This is becomes a problem when you have hundreds of microservices that need to communicate with each other.

The naive solution is to use a different url per service, and pass those in from the outside: this means that every time a single microservice address changes, all its consumers need to be updated. Things get even more complicated if we have several instances of a microservice that we want to consume: what if the one we know the address of is offline, but we have some other up and running?
The standard solution to this problem is a discovery micro service, that points us to the service we are asking for. Furthermore, to help other developers explore the service, a new standard has been suggested: adding a  `/disco` endpoint to each microservice, listing the available endpoints and a short description of each one of them. [2]s

### Handling Failures 

Being resilient to failures is a good property for a system, but it raises the problem of identifying when one of the many microservices shuts down, and not hammering it until it’s restarted. Both problems can be solved using the circuit breaker pattern, explored in detail [in this blog post](http://martinfowler.com/bliki/CircuitBreaker.html) by Martin Fowler.

## Final thoughts

Microservices are not the final goal: as developers, that remains building software that satisfies the business requirements. But they are a very powerful tool we have in our arsenal, and that can help us reaching the goal, and building overall better software.

References

[1] : Conway, Melvin E. (April 1968), "[How do Committees Invent?](http://www.melconway.com/research/committees.html)", [Datamation](http://en.wikipedia.org/wiki/Datamation) 14

[2] : Greg Young, “The future of Microservices”, µCon 2014
