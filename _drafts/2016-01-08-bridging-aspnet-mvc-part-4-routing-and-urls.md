---
title: Bridging ASP.NET MVC - Routing and URL Generation
layout: post
---

This is **part 4** of a multi-part series on "fixing" some of the inherit design problems with ASP.NET MVC.

- [Part 1 - Introduction]({% post_url 2016-01-05-bridging-aspnet-mvc-part-1 %})
- [Part 2 - The View Engine]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %})
- [Part 3 - View Model Conventions]({% post_url 2016-01-07-bridging-aspnet-mvc-part-3-view-model-conventions %})
- [Part 4 - Routing and URL Generation](2016-01-07-bridging-aspnet-mvc-part-4-routing-and-urls)
- Part 5 - The Controller Factory
- Part 6 - ActionResults and Content Negotiation

A common theme of this series is derived from the following quote I once read (but can no longer attribute):

> Don't let your routing solution dictate how to organize your code.

In essence, **don't name your controllers and actions based on what you want to be the mapped URL**.

ASP.NET MVC is built upon `System.Web.Routing`, a powerful abstraction and HttpModule that maps URLs to instances of an `IRouteHandler`. Effectively `System.Web.Mvc` is just that.

Among the many poor defaults and other conventions established in ASP.NET MVC is the "default route" code that you usually find in the baked-in project template that almost everyone uses:

    routes.MapRoute(
        "Default",
        "{controller}/{action}/{id}",
        new { controller = "Home",
              action = "Index",
              id = "" }
    );

As we know this establishes the convention of a controller and action name defining the mapped URL. And, like many of these conventions, it is directly "borrowed" from Rails.

In the early days of ASP.NET MVC you sort of had to follow this convention because it was a pain to map URLs otherwise. But soon came several open-source projects trying to fix that. My favorite was [AttributeRouting](https://github.com/mccalltd/AttributeRouting): you use simple attributes on your class and methods to define the URL, or define your own conventions, and the framework wires it all up. You get a really good programming model and good decoupling.

In fact this library was so good and popular that Microsoft effectively reimplemented parts of AttributeRouting and baked it right into the framework as of version 5! (Sad fact: they only implemented a fraction of AttributeRouting's features.)

### Tip: Use Attribute Routing, Exclusively.

While I definitely recommend using attribute based routing exclusively, there are some gotchas. I recommend reading about some of the performance issues that can pop up. Here is a good in-depth article that talks about it: [https://samsaffron.com/archive/2011/10/13/optimising-asp-net-mvc3-routing](https://samsaffron.com/archive/2011/10/13/optimising-asp-net-mvc3-routing).

### Tip: Avoid String Constants for URL Generation

As an aside, I really dislike that most of the `UrlHelper` methods require strings for controller and action names. I encourage the use of `Microsoft.Web.Mvc`, which includes a special class called `ExpressionHelper` that can generate a URL string from a lambda expression referencing a controller class and action method. And so that I don't need to pollute my project with namespace references, I created a small set of extension methods to abstract away that mess:

<script src="https://gist.github.com/jdaigle/453bcee73d8ee0ab7a7d.js"></script>

Code that needs to generate a URL is now quite clean. It's easier to refactor and to find references via static analysis.

    var link
        = this.GetURL<SomeController>(c => c.SomeAction(params...));

*Be warned that you do pay a small performance penalty with this approach.*

You can also take a look at [T4MVC](https://github.com/T4MVC/T4MVC). This takes a different approach of using [T4 text templates](https://msdn.microsoft.com/en-us/library/bb126445.aspx) to generate constants and static helper methods based on your project and source code. I have used this extensively in the past which great success. It's fast because you're using constants. But it's a bit intrusive since it requires changing your controller methods to be virtual (not to mention the whole code-gen part that some people dislike).

### Who uses Areas?

