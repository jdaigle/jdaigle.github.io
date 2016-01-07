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

Don't let your routing solution dictate how to organize your code.

### Tip: Avoid String Constants for URL Generation

As an aside, I really dislike that most of the `UrlHelper` methods require strings for controller and action names. I encourage the use of `Microsoft.Web.Mvc`, which includes a special class called `ExpressionHelper` that can generate a URL string from a lambda expression referencing a controller class and action method. And so that I don't need to pollute my project with namespace references, I created a small set of extension methods to abstract away that mess:

<script src="https://gist.github.com/jdaigle/453bcee73d8ee0ab7a7d.js"></script>

Code that needs to generate a URL is now quite clean. It's easier to refactor and to find references via static analysis.

    var link
        = this.GetURL<SomeController>(c => c.SomeAction(params...));
