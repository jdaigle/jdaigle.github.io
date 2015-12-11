---
title: ASP.NET Identity - A Case Study in Monolithic Frameworks  
layout: post
---

## Abstract

**[ASP.NET Identity](https://aspnetidentity.codeplex.com)** is a monolithic membership framework, especially when combined with **Microsoft.OWIN.Security** (which is almost a requirement to successfully using it!). As a monolithic framework, the developer eventually suffers when trying to plugin or extend the framework to do something that is no supported out of the box. Developers may also struggle simply using what's provided *in the box* due to severe leaky abstractions.

## Separation of Concerns

First I want to briefly cover the difference between *authentication* and "*membership*" in web application.

**Authentication** specifically refers the mechanism by which a user's identity is determined from an HTTP request. These days it's quite common to use an HTTP cookie that serializes some sort of identity; be it a session or user ID, and perhaps identity claims. Often the cookie is encrypted so that only the server application is able to read and/or modify.

Other authentication mechanisms include [basic access authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) (the username/password are encoded into the HTTP header of every single request), or tokens which are often included in an HTTP header or query string parameter.

In the ASP.NET world the term **membership** refers to the mechanism by which users are managed. Be it a database or a directory. "Authentication" is sometimes overloaded to refer to the process of validating a token or set of credentials against the user's membership. This usually occurs during *login* and frequently also occurs on each HTTP request to validate the authentication token.

To make things a more complicated, modern web applications often do not directly deal with user credentials. Instead relying on external authentication via OAuth or SAML (think Facebook, Google, Microsoft, ADFS, etc.). But for this article, they're slightly out of scope.

## Abridged History of Membership and Authentication in ASP.NET

Most of what's existed since the first version of ASP.NET exists as part of `System.Web.Security`. That means it's part of the BCL and shipped with every installation.

One quick note: some authentication such as Basic and Windows auth are actually implemented natively by IIS and aren't part of the .NET framework.

Here's a pretty good article that covers the old membership providers: [http://brockallen.com/2012/09/02/think-twice-about-using-membershipprovider-and-simplemembership/](http://brockallen.com/2012/09/02/think-twice-about-using-membershipprovider-and-simplemembership/). It also offers a critique which aligns with the goal of this article.

## Enter ASP.NET Identity

Again, to piggy back on good writings, here is Brock Allen again: [http://brockallen.com/2013/10/20/the-good-the-bad-and-the-ugly-of-asp-net-identity/](http://brockallen.com/2013/10/20/the-good-the-bad-and-the-ugly-of-asp-net-identity/).

## Leaky Abstractions

Here's one [egregious example](https://aspnetidentity.codeplex.com/workitem/2540). The `UserManager` class exposes `IQueryable<TUser> Users`. In this example the developer is using `Microsoft.AspNet.Identity.EntityFramework` to provide a concrete implementation for `TUser`. However the developer must understand how EntityFramework works and how the entities are implemented in order to use `IQueryable<TUser>` correctly.

I should also point out that the implementation of `IQueryable<TUser> Users` on `UserManager` throws a `NotSupportedException` if the `IUserStore<TUser>` implementation does not implement the correct interface! This is another perfect example of a leaky abstraction where the developer must understand the implementation in order to use the library.

## Missing Features and Abandonware

Brock Allen pretty much [nails it again](http://brockallen.com/2013/10/20/the-good-the-bad-and-the-ugly-of-asp-net-identity/#ugly) talking about so many security and identity related concerns that the framework simply does not handle.

I find this quote particularly interesting:

>  Of course, with this redesign, I think Microsoft is in a much better place to add these features in a future release. But it ain’t there now.

Allen's article was written over two years ago and new features really *have not* been added!

**Using these monolithic frameworks often results in the developer writing much more "glue" code to get things working that it would take to simply write a non-abstracted version of what they're trying to accomplish**. These abstractions allow you to do neat things for demos, such as quickly building a working web application from templates. But for real world production applications? It can get rough.

I should also point that the current version of the library is effectively **abandonware**. There are no new updates in the current repository. In fact the repository has [moved to GitHub](https://github.com/aspnet/Identity) to be included in ASP.NET 5. But I would like to point out that this is effectively a "port" of current version (as [is evident](https://github.com/aspnet/Identity/commit/aa4787cc67ed02ba0d2c08adecf8ada75b2d7239)) since ASP.NET 5 is built on completely new tooling.

## Who Uses This Stuff?

I honestly wonder. Outside of small proof-of-concept or very small line-of-business applications, does anyone use these frameworks?

My guess is no. Because at the end of the day it's almost easier to roll your own. You'll get better performance and you can make it easier for junior developers to consume your API when building new application features. But this comes with an important caveat: **security is hard**. And **you should not be building your own authentication and security libraries unless you actually know what you're doing**. So it's a tough choice for a small shop that doesn't have the expertise.

## Next

In a future post I will talk about the [OWIN security components](http://katanaproject.codeplex.com/). These deal with cookie authentication, and provide implementations for external login providers (i.e. OAuth). But as a preview that library shares most of the same criticisms about monolithic frameworks, leaky abstractions, and more. 