---
layout: post
title: "Bede gaming at NDC 2014"
date: 2014-12-29 14:00:00 +0100
author: Richard Forrest
comments: true
categories: identity, accesscontrol, openid, oauth, oath2, sts 

---

User identity and how we control access to systems has change a lot over the last decade or so, especially in the .net space.   At Bede we are looking to provide some significant updates to our authentication and access control services.  With those goals in in mind a couple of members of our platform services team attended an Identity workshop at ndc London in December hosted by [Brock Allen](http://brockallen.com/) and [Dominick Baier](http://leastprivilege.com/) here are some of the topic areas we covered.

<!-- more -->
##First Some History.. 

Back in 2002 with the first release of .net and ASP.net Identity and authentication in our applications basically boiled down to these two interfaces

    interface IIdentity
	{
    	bool IsAuthenticated {get;}
		string AuthentictaionType {get;}
		string Name {get;}
	}

	interace IPrinipal
	{
		IIdentity Identity {get;}
		bool IsInRole(string roleName);
	}   

This was how we secured our UI or applications using the couple of inbuilt authentication mechanisms provided by the .net framework e.g. Windows auth or Asp.net forms authentication.  These mechanisms handled validating user credentials, making identities (WindowsIdentity and FormIdentity respectively) and principals of the relevant type for us and popping them on the Thread.CurrentPrincipal.  Happy days problem solved move on... right? Well, still have a huge disconnect between how we handled security in our applications and in our services.  On the front end we may have been using forms authentication while our service used something like [wsSecurity](http://en.wikipedia.org/wiki/WS-Security). 

Then our applications started getting more complicated - our users may log into our web applications, which in turn access back end services (which we also want to secure), but they can also use native client or smart phone applications.  These native apps may connect directly to our backend service or via public APIs.  We may also want third party applications to access subsets of our users data or allow users to log into our systems using there existing social media accounts.

{% img center /images/sts-arch.png 'Typical Application authentication Scenario' 'Typical Application authentication Scenario' %}

The solution is to have a single place where we handle user authentication, and manage user identities using a standard set of protocols that all our different clients can understand, that's where OpenID Connect comes in.  
 
##OpenID Connect

I bet your asking yourself is this not the problem that [SAML](http://saml.xml.org/) was meant to solve (well maybe only some of you are asking yourself that). Yes and it works.  But you have to keep in mind

> "SAML is the windows XP of Identity" \[1\]

* It was built 12 years ago.
* It's no longer developed.
* Noone works on it.
* Its not the future.

Well then, why can't we just use OAuth2? That's popular right?  Well yes, but OAuth2 is, as I'm sure we all know, [not an authentication protocol](http://oauth.net/articles/authentication/) its just a protocol for handling API access tokens. Lots of companies have tried to turn OAuth2 into an authentication protocol which is not a trivial problem.  The result of this is that we now have a lot of organisation with their own [flavor](https://oauth.io/providers) of OAuth2.  Now you have facebook style OAuth and twitter style OAuth and we end up needing a client library for each. This trend of role your own authentication protocol has resulted in some fairly high profile hacks over the last few years.

So what's [OpenID Connect](http://openid.net/connect/) well

>"OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol."

This allows us to use OAuth2 for authentication by:

* Defines authentication protocol on top of OAuth2. Whereas OAuth2 is primarily used for API access tokens, OpenID defines an identity token that can be used to identify a specific user.  
* Defines a standard [token type](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) which OAuth does not do, making it interoperable.
* Defines standard cryptography that tells us how our tokens are protected - no need to roll your own.
* Defines how to validate a token.
* Defines work flows for native applications, browsers and servers.

This means we can now do authentication and API access control in a single protocol.

##Identity Server

Thinktecture [IdentityServer](https://github.com/thinktecture/Thinktecture.IdentityServer.v3) is an open source OpenID Connect secure token server implementation. There are a host of non .net [implementations](http://openid.net/developers/libraries/) as well. Secure token servers have three major endpoints: 

* **Authorise** provide the pages the user is redirected to to allow them to login, it is basically responsible for rendering the UI
* **Token** deal with the identity and access control tokens
* **User info** give additional user details, like a profile 

##Flows

OpenID connect supports different workflows that allow you to authenticate different types of application.

* **Implicit Flow** for native/browser/web apps user is redirected to login   
* **Authorisation Code Flow** for server based applications
* **Hybrid Flow** for applications combining both

The simplest example is probably the implicit flow.  A user wishes to log into a web application, the application redirects the user to our authorisation server with this query string

	GET /authorize
	?client_id=app1
	&redirect_uri=https://app1.com/login
	&response_type=id_token
	&response_mode=form_post
	&scope=openid email

This identifies the application, where we want the user redirected to when they have authenticated, what response we are looking for - in this case a token, how we want that response returned - here as a form post, and the scope of information we want to know about the user - here we are asking for their identity (*"openid"*) and their email. 

The user will be asked to authenticate somehow by our secure token server.  This could be done by the user entering a username and password which is validated by the secure token server, or else using some third party social media site.  They may then be asked to give consent for the scopes we have requested and when that is complete a token containing the information requested will be posted back to us. 

Using the same set of protocols and secure token server we can request access tokens for our API so that we can control access to our services. 

##What's in the token

The token format we get back from our client is a [json web token](http://jwt.io/).  These token are a set of users claims represented as a json object that's base64 encoded and then digitally signed. It provides a URL safe way of transferring user identities in a format that can be validated by any client. 

A claim is just a statement about the user:

> "bob's email is bob@gmail.com" 

or

> "bob is an admin" 
 
they are usually represented as key value pairs e.g. "email":"bob@gmail.com".

The claims in the json web token are part of user the identity information that we have request and is determined by the scope we asked for.

**Example claims**

	{
	  "sub": 1234567890,
	  "name": "John Doe",
	  "email": "Johndoe@email.com"	  
	}

"sub" in the above example is the unique identifier for a specific user this will be provided when we request the openid scope. 

##What this means for my application

We now have a set of protocols that allow us to manage user identity and access control for a range of application and client types from one single secure token server.  In the .net space we still have the two interfaces we are familiar with IIdentity and IPrincipal but now we can inherit from a common set of claims based implementation.

	class ClaimsIdentity : IIdentity
	{
		IEnumerable<Claim> Claim {get;}
	}

	class ClaimsPrincipal : IPrincipal
	{
		ReadOnlyCollection<ClaimsIdentity> Identities {get;}
	}   

All our users claims are provided in the openid token given to us by our secure token server.      

References

\[1\]: Craig Burton 
