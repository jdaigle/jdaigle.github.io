---
title: Bridging ASP.NET MVC - Part 4 - Routing and URL Generation
layout: post
date: 2016-01-13
---

This is **part 4** of a multi-part series on "fixing" some of the inherit design problems with ASP.NET MVC.

- [Part 1 - Introduction]({% post_url 2016-01-05-bridging-aspnet-mvc-part-1 %})
- [Part 2 - The View Engine]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %})
- [Part 3 - View Model Conventions]({% post_url 2016-01-07-bridging-aspnet-mvc-part-3-view-model-conventions %})
- [Part 4 - Routing and URL Generation]({% post_url 2016-01-13-bridging-aspnet-mvc-part-4-routing-and-urls %})
- Part 5 - The Controller Factory
- Part 6 - ActionResults and Content Negotiation

A common theme of this series is derived from the following quote I once read (but can no longer attribute):

> Don't let your routing solution dictate how to organize your code.

In essence, **don't name your controllers and actions based on what your URLs to look like**.

ASP.NET MVC is built upon `System.Web.Routing`, a powerful HttpModule and abstraction that maps a request (by matching URL, HTTP method, and other headers) to instances of an `IRouteHandler`. Effectively `System.Web.Mvc` is just that.

Among the many poor defaults and other conventions established in ASP.NET MVC is the "default route" code that you usually find in the baked-in project template that almost everyone uses:

    routes.MapRoute(
        "Default",
        "{controller}/{action}/{id}",
        new { controller = "Home",
              action = "Index",
              id = "" }
    );

As we know this establishes the convention of a controller and action name defining the mapped URL. And, like many of these conventions, it is directly "borrowed" from Rails. We even have the silly notion of a "Home" and "Index" which I sort of consider "legacy" HTTP terms.

In the early days of ASP.NET MVC *you sort of had to follow this convention* because it was a pain to otherwise map URLs. But eventually we saw several open source projects attempt to fix that. **My favorite was [AttributeRouting](https://github.com/mccalltd/AttributeRouting)**: you use simple attributes on your classes and methods to define the URL, or define your own conventions, and the framework wires it all up. You get a really good programming model and good decoupling of code and URL paths.

In fact this library was so good, and popular, that Microsoft effectively re-implemented parts of *AttributeRouting*; baking it right into the framework!

But there's a sad post-script: they only implemented a fraction of *AttributeRouting*'s extensive list of features. And they did so in a way that you can't really use *AttributeRouting* as-is in ASP.NET MVC 5. And the project is now pretty much abandoned.

### Tip: Use Attribute-Based Routing, Exclusively.

While I definitely **recommend using attribute-based routing exclusively**, there are some things to watch out for especially as you start adding a lot of mapped routes. I recommend reading about some of the performance issues that can pop up. Here is a good in-depth article that talks about it: [https://samsaffron.com/archive/2011/10/13/optimising-asp-net-mvc3-routing](https://samsaffron.com/archive/2011/10/13/optimising-asp-net-mvc3-routing).

**And don't name your routes.** Yes, it will greatly speed up URL generation. But unless you have a relatively simple application that doesn't change frequently, you're going to have to *maintain* both those names and their usage.

### Tip: Avoid String Constants for URL Generation

I really dislike that most of the `UrlHelper` methods require strings for controller and action names. I encourage the use of `Microsoft.Web.Mvc`, which includes a special class called `ExpressionHelper` that can generate a URL string from a lambda expression referencing a controller class and action method. And so that I don't need to pollute my project with namespace references, I created a small set of extension methods to abstract away that mess:

<script src="https://gist.github.com/jdaigle/453bcee73d8ee0ab7a7d.js"></script>

Code that needs to generate a URL is now quite clean. It's easier to refactor and to find references via static analysis.

    var link
        = this.GetURL<SomeController>(c => c.SomeAction(params...));

*Be warned that you do pay a small performance penalty with `ExpressionHelper`.* Your mileage may vary. So do your own testing and performance profiling!


**You can also take a look at [T4MVC](https://github.com/T4MVC/T4MVC)**. This takes a different approach that utilizes [T4 text templates](https://msdn.microsoft.com/en-us/library/bb126445.aspx) to generate constants and static helper methods based on your project and source code. I have used this extensively in the past which great success. It's fast because you're using constants. But it's a bit intrusive since it requires changing your controller methods to be virtual and creates messy overloads (not to mention the whole code-gen part that some people dislike).

#### Experimental Idea Ahead

An approach that I'm beginning to experiment with falls somewhere in between full-on expressions and static helpers. This article on [Delegate-based strongly-typed URL generation in ASP.NET MVC](http://maxtoroq.github.io/2013/04/delegate-based-strongly-typed-url.html) shows how we can use an action method delegate to generate route values (action name, parameters, etc.).

But since action methods are instance methods, this approach requires a controller instance. Which, as the article explains, can get tricky if we want an action delegate for a controller *other than the one currently executing*. It's not too hard to get a "dummy", or "uninitialized", instance like this:

        public static T InstanceOf<T>() where T : ControllerBase
        {
            return System.Runtime.Serialization.FormatterServices.GetUninitializedObject(type);
        }

One could even borrow from the concepts of T4MVC and automatically generate a static helper class with these "uninitialized" controller instances cached for URL generation. But so far any API I've created feels too messy compared to a lambda expression or T4MVC's overrides. 

More experimentation is needed.

### Tip: Avoid Areas

You can almost think of an area as a little MVC application embedded into another. It adds a new route value for matching named "area". Often by convention the area name is simply another part of the URL path that precedes everything else. The built-in view engines and controller factory are also aware of areas, so you end up with self-contained "Controllers/" and "Views/" namespaces/folders under the area. 

**But it's the completely wrong approach to modularity.**

In a future article I plan on discussing some alternatives that actually make sense.

### Further Reading and References

* [Optimizing ASP.NET MVC3 Routing](https://samsaffron.com/archive/2011/10/13/optimising-asp-net-mvc3-routing)
* [Why ASP.NET MVC Routing Sucks](https://maxtoroq.github.io/2014/02/why-aspnet-mvc-routing-sucks.html) - (Note that I don't agree with all of Max's conclusions)
* [Delegate-based strongly-typed URL generation in ASP.NET MVC](http://maxtoroq.github.io/2013/04/delegate-based-strongly-typed-url.html)