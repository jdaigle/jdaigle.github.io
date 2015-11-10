---
title: OWIN, Can Someone Explain This to Me?
layout: post
---

Recently I started diving into [SignalR](http://signalr.net/) which quickly led me down the rabbit-whole of learning about OWIN. I've been developing ASP.NET applications for over a decade and I figured I had a firm grasp the concepts. But in the last few years Microsoft has really been disturbing the status quo with new frameworks and patterns.

And [OWIN](http://owin.org/) is the perfect place to start exploring.

### First a bit of history

**ASP.NET** has been around since the beginning of the .NET framework, around 2002. It's the successor to Microsoft's ASP framework for server-side scripting. Beginning primarily has WebForms (ASPX pages), it has really matured over the past thirteen years.

If you've ever built something that references **System.Web.dll**, you're using ASP.NET.

But it's a huge **monolithic framework**. And it's heavily dependent on IIS and Windows to host an application. IIS itself isn't really a problem; it's arguably one of the best web servers available. Certainly the best available for Windows. The problem is that ASP.NET brings with it **a lot of baggage that's incredibly difficult to decouple**.

For example, if you're building a modern web application you're probably not using session state. Yet that module is loaded and runs with every request. The original authentication module, will still useful in some cases, is very outdated. The error handling module is... terrible (note to self, write about modern ASP.NET error handling).

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Overheard &quot;IIS is the fastest web server, as long as you don&#39;t load System.Web&quot; <a href="https://twitter.com/hashtag/ndclondon?src=hash">#ndclondon</a></p>&mdash; Arjan Einbu (@aeinbu) <a href="https://twitter.com/aeinbu/status/407816285058514944">December 3, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### Enter OWIN

Here is a pretty neat history of how OWIN came about: [http://www.slideshare.net/panesofglass/a-brief-history-of-owin](http://www.slideshare.net/panesofglass/a-brief-history-of-owin).

[OWIN](http://owin.org/) stands for **Open Web Interface for .NET**. It is simply a specification for how a server should expose an HTTP request/response to a software module. The specification allows high level of abstraction from the actually HTTP host and server.

The [specification](http://owin.org/html/spec/owin-1.0.html) itself is rather concise. Here's the 10,000 foot view:

* A **Server** implements HTTP, running in some **Host** process, and exposes an "Environment" for the web application.
* An **environment** is modeled as just an `IDictionary<string, object>`. Strings are keys which look up something about the request execution. I.E. parts of the request or the response.
* The specification defines many of these keys and what type of object should be returned.
* Software components are built as **Middleware** which are registered with the server. Multiple Middleware components can be strung together to build a pipeline. 
* The **AppFunc** delegate is the primary interface in OWIN. Each Middleware component must implement a method that matches this delegate: `Func<IDictionary<string, object>, Task>`. It's a function that accepts the "environment" dictionary as a parameter, and *returns a Task*. This means that **OWIN is inherently async**!

And that's pretty much it.

### Break out the Katana 刀

No not [刀](https://en.wikipedia.org/wiki/Katana), the Japanese samurai sword. 

**[Katana](http://katanaproject.codeplex.com/)** is Microsoft's official implementation of OWIN. It's an open source project on codeplex made of up several components.

Many of these components provide convenience classes that expose an API similar to ASP.NET. Such as an *OwinContext*, *OwinRequest*, *OwinResponse*, etc.

Katana also delivers, presently, two different ways to host an OWIN application.

1. You can easily **self-host** an HTTP server that runs OWIN middleware. It's kind of just a wrapper around **System.Net.HttpListener** (i.e. http.sys) that implements the OWIN spec. This allows you to host an OWIN based application where you may not have access to IIS, or if you wanted to embed it in another application or service.

2. It can run side-by-side with ASP.NET using **Microsoft.Owin.Host.SystemWeb**. Technically it's adding OWIN as an ASP.NET module, so you're still getting all of that baggage. But, in addition to hosting in IIS, it allows you mix OWIN middleware components with existing ASP.NET components and frameworks such as ASP.NET MVC.

There is a third way to host an OWIN application. You can actually host OWIN in IIS *without* System.Web using a project named [Helios](https://www.nuget.org/packages/Microsoft.Owin.Host.IIS/). This is an experimental implementation that is technically **not supported**. And it's *not open source* from what I can tell. It essentially does a bunch of Interop magic to launch the code in IIS without ASP.NET.

Developing an OWIN application or middleware component is actually relatively straight forward too.

For an application:

1. Start with a "startup" class which a method that matches this name and signature: `public void Configuration(IAppBuilder app)`. Katana's hosting libraries use conventions to find this method and call it.
2. Using IAppBuilder, register middleware components.

There are various extensions methods on IAppBuilder which register middleware components.

* You can call `Run()` and pass in a `Func<IOwinContext, Task>`. Notice that this is similar to the **AppFunc** delegate in the spec. But for convenience the "environment" is now a strongly typed interface.
* You can call `Use()` and pass in a `Func<IOwinContext, Func<Task>, Task>`. This is similar, gives you *the next task* in the pipeline to execute. This enables you inject logic around subsequent middleware.
* Middleware can also be implemented as a "module". There simply needs to be a class which implements a specific convention based on the core API.

Middleware can also be registered to specific paths. This enables interesting scenarios such as branched pipelines; where for a particular path a different set of

Now, it seems that Katana in it's current form is basically "done". The next version will officially be part of **ASP.NET vNext** which is currently still being developed. **It is known that there will be breaking API changes between the current release and ASP.NET vNext**. But, more importantly, the general programming model and patterns will remain relatively unchanged. The good news is that it should not take a lot of effort to port existing OWIN applications and middleware to vNext.

Katana
- MS implementation of OWIN spec
- Includes IIS or self-host
- Convience classes (OwinContext/OwinRequest/etc)
-- similiar API to System.Web's HTTPContext/Request/Response
- Set of middleware for common features - authentication, static content
-- This will be how MS will deliver new features for ASP.NET

OWIN Defines an Architecture
- Host: It's the "process".
- Server: implements HTTP and the OWIN API.
- Middleware: plugs into Server via registration.

Programming Model
- Requires startup class with: public void Configuration(IAppBuilder app)
- Register middleware in IAppBuilder
-- either via Run() which accepts Func<IOwinContext, Task> as the handler
-- or Use() which accepts Func<IOwinContext, Func<Task>, Task> which allows you to insert a handler tha calls Next();
-- or use Middleware "module". A class that implements a convention based on core API
-- middleware can be mapped to a specific path, allows for branched pipeline with separate configuration
-- Base References: Owin, Microsoft.Owin
-- IIS Host: Microsoft.Owin.Host.SystemWeb
