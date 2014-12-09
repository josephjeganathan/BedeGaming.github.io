---
layout: post
title: "Adding Request Scope for Non-Web Applications"
date: 2014-12-09 17:02:00 +0100
comments: true
author: Paul Devenney
categories: code,autofac,.NET
---

Here at Bede we use [Autofac](http://autofac.org/) a lot. It's an excellent [IoC](http://en.wikipedia.org/wiki/Inversion_of_control) container which allows us to create well structured code focused on interfaces rather than implementations. If you are not familiar with IoC, or it's most commonly used form of [Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection), I'd suggest the essential (.NET) reading of [Dependency Injection in .NET](http://www.amazon.co.uk/Dependency-Injection-NET-Mark-Seemann/dp/1935182501) as a primer.

A typical basic example would be at the beginning of our application (called the "composition root") to do something like 

    var builder = new ContainerBuilder();
    builder.RegisterType<MyWorkService>().As<IWorkService>();
    _container = builder.Build();

This says that any time someone asks for an ```IWorkService``` that they should have that satisfied with a ```MyWorkService``` implementation. This leaves you free to write code like

    public class MyController : ApiController
    {
		private readonly IWorkService _workService;

		public ProfilesController(IWorkService workService)
		{
			_workService  = workService;
		}
	}

...and not be worried about where that ```IWorkService``` is going to come from. It's a critical pattern for clean, re-usable, testable code, and one we use consistently throughout the business in all our apps.


##IoC Containers and the Web - Instance Per Request
IoC containers like Autofac come ready prepared to hook into web and API applications (my example controller above would fail to run if I had not hooked up Autofac). We find ourselves naturally starting to depend upon Autofacs built in help for scoping dependencies. For example if we changed the ```IWorkService``` registration above to 

    var builder = new ContainerBuilder();
    builder.RegisterType<MyWorkService>().As<IWorkService>().InstancePerRequest();
    _container = builder.Build();

We actually change _when_ a new ```MyWorkService``` is constructed. Before it would be _any time_ a contructor asked for an ```IWorkService```. Now it is only once for each web request. We could go a step further and do 

    var builder = new ContainerBuilder();
    builder.RegisterType<MyWorkService>().As<IWorkService>().SingleInstance();
    _container = builder.Build();

...and now we will only ever construct one object and hand it out every time we are asked to satisfy the ```IWorkService``` class.

These techniques are excellent - because it allows us to ensure that objects that are "costly" to instantiate are only constructed when absolutely needed, and further, items that are created once "per request" in a web application can essentially persist context of that request within the service (username, request parameters, caller IP Address, etc.)

We leverage the "InstancePerRequest" for several contextual services, as it means that you can assign context values to service properties, rather than as parameters of its methods. For example:

	public ProfilesController(IWorkService workService)
	{
		_workService  = workService;
	}

	public HttpResponseMessage Update(Profile profile)
	{
		List<string> values;
		Request.Headers.TryGetValues("userAppKey", out values)
		_workService.AppKey = values[0];

		...

		_workService.DoStuff(profile);
		_workService.DoMoreStuff();
	}

This is great, as we now don't have to add the "userAppKey" to each and every method signature.

##Instance Per Request for non-web applications
< other times you want to use this - for example message event handling >


