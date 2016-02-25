---
title: Integrating ASP.NET MVC and WebAPI
layout: post
---

The *current* versions of ASP.NET MVC and WebAPI (5.x) exists as two separate frameworks. Each has it's own routing conventions, model binding, action filters, dependency resolver, controller factory, configuration patterns, etc. And if you dig into the [source code](https://aspnetwebstack.codeplex.com/SourceControl/latest) you'll find A LOT over overlapping code; it's practically copy/paste between the namespaces.

But this does not mean you cannot mix the two together.

[Scott Allen](http://odetocode.com/about/scott-allen) has an [excellent article](http://odetocode.com/blogs/scott/archive/2013/07/01/on-the-coexistence-of-asp-net-mvc-and-webapi.aspx) in which he describes when to use each framework and when it might be appropriate to use both in the same project.

> I’ve gotten more than a few questions over the last year on how to use the ASP.NET MVC framework and the Web API framework together. Do they work together? Should they work together? When should you use one or the other?
> 
> Here’s some general rules of thumb I use.

Because each framework has it's own separate processing pipeline, you need to consider carefully whether you *really need* both frameworks in the same project. Especially if you do important things with routing and action filters, such as authentication or authorization - it'll take extra work to share code and algorithms between the two.

Fortunately the future is bright. ASP.NET Core promises to finally unify the frameworks.