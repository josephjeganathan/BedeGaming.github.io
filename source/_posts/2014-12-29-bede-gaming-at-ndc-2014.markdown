---
layout: post
title: "Bede gaming at NDC 2014"
date: 2014-12-29 14:00:00 +0100
author: Richard Forrest
comments: true
categories: identity, access control, openid, oauth 

---

User identity and how we control access to systems has change a lot over the last decade or so, especially in the .net space.   At Bede we are looking to provide some significant updates to our authentication and access control services.  With those goals in in mind a couple members of our platform services team attended an Identity workshop at ndc London in December hosted by [Brock Allen](http://brockallen.com/) and [Dominick Baier](http://leastprivilege.com/) here are some of the topic areas we covered.



##Some history

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

This was how we secured our UI or applications using the couple of core authentication mechanism provided Windows or Asp.net forms authentication.  These mechanism both handled validating user credentials, making identity and principals of the relevant type for us and popping them on the Thread.CurrentPrincipal.  Happy days problem solved move on right? 

Well we did have a huge disconnect between how we handled security in our applications and in our services.  On the front end we may have been using forms authentication while our service used something like wsSecurity. 

Then our application started getting more complicated our users may log into our web applications which in turn accesses back end services (which we also want to secure), but they can also use native client or smart phone applications.  These native apps may connect directly to our backend service or via public APIs.  We may also want third party applications to access subsets of our users data or allow users to log into our systems using there existing social media accounts.

{% img center /images/Typical Application.png 'Typical Application authentication Scenario' 'Typical Application authentication Scenario' %}

The solution is that we have a single place where we handle user authentication and manage user identities using a standard set of protocols that all our different clients can understand and that's where OpenID Connect comes in.  

 
##OpenID Connect

I bet your asking yourself is this not the problem that [SAML](http://saml.xml.org/) was meant to solve. Yes and it works.  But 

> "SAML is the windows XP of Identity" [1]

* It was built 12 years ago
* Its no longer developed
* No body works on it 
* Its not the future

Why can't we just use OAuth2 that popular now right well yes but OAuth2 is [not an authentication protocol](http://oauth.net/articles/authentication/) its just a protocol for distributing API access tokens. Lots of companies have tried to turn OAuth2 into an authentication protocol which is not a trivial problem.  The result of this is that we now have a lot of organisation with their own [flavor](https://oauth.io/providers) of OAuth2.  Now you have facebook style OAuth and twitter style OAuth and we end up needing a client library for each. This trend of role your own authentication protocol has resulted in some fairly high profile hacks over the last few years.

So whats [OpenID Connect](http://openid.net/connect/) well

>"OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol."

What it does is to allow us to use OAuth2 for authentication by:

* Defines authentication protocol on top of OAuth2. 
Where as OAuth2 is primarily used for AIP access token OpenID defines an identity token that can be used to identify a specific user.  
* Defines a standard [token types](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) which OAuth does not do, which makes it interoperable.
* Defines standard cryptography that tell us how our token are protected no more role your own.
* Defines how you validate a token
* Defines workflows for native applications, browsers and servers.

This means we can now do authentication and API access control in a single protocol.

##Identity Server

Thinktecture [IdentityServer](https://github.com/thinktecture/Thinktecture.IdentityServer.v3) is an open source OpenID Connect secure token server implementation.  Secure token server have three major endpoints: 

* **Authorise** provide the pages the user is redirected to to allow them to login, it basically responsible for rendering the UI
* **Token** deal with the identity and access control tokens
* **User info** give additional user detail like a profile 

###Flows

There a different workflows supported by OpenID connect and Identity server

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

This identifies the application, where we want the user redirected to when they have authenticated, what response we are looking for in this case a token, how we want it back this time as a form post and the scope of information we want to know about the user here we are asking for their identity (*"openid"*) and their email. 

The user will be asked to authenticate by our secure token server they may be asked to give consent for the scopes we have requested and then the token containing the information requested will be posted back to us.  

References

[1]: Craig Burton 