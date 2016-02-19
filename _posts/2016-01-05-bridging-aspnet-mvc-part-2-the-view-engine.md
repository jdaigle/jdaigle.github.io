---
title: Aftermarket ASP.NET MVC - Part 2 - The View Engine
layout: post
date: 2016-01-05 09:00
---

This is **part 2** of a multi-part series on "fixing" some of the inherit design problems with ASP.NET MVC.

- [Part 1 - Introduction]({% post_url 2016-01-05-bridging-aspnet-mvc-part-1 %})
- [Part 2 - The View Engine]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %})
- [Part 3 - View Model Conventions]({% post_url 2016-01-07-bridging-aspnet-mvc-part-3-view-model-conventions %})
- [Part 4 - Routing and URL Generation]({% post_url 2016-01-13-bridging-aspnet-mvc-part-4-routing-and-urls %})
- [Part 5 - Feature Folders]({% post_url 2016-02-19-aftermarket-aspnet-mvc-part-5-feature-folders %})
- Part 6 - ActionResults and Content Negotiation

Update 2016-02-18: I've also setup a [repository on GitHub](https://github.com/jdaigle/aspnetmvc5demo) that includes many of these experiments and implementations.

The problem with the default view engine in ASP.NET MVC stems not from how it renders a view, but how it *selects and finds* a view to render. If you dig through the source code you'll discover that *selecting and finding* a view to render is tightly coupled with the actual view engine itself. That can be troublesome if we want to implement our own conventions.

Further, **the built-in view engine uses a convention of selecting and finding a view based on the controller and action's name.** I consider this an anti-pattern. It couples a view to controllers and action methods by name. It is possible to override this behavior by constructing a ViewResult with the name or path of the specific view you want selected. But again we're coupled, since the convention is to look in a directory based on the name of controller. We also have to use string constants throughout which means we lose the ability to refactor and perform static code analysis.

**I think a better design is to select and find the view based on the type of view model returned.** I'll talk about this more in a future article, but a really good convention to adopt is having a strongly typed view model per view. I also recommend that the view model class' namespace match the "path" of the view. In fact, just put it in the same directory as the view! A benefit of this convention is that we can select the view simply by inspecting the type of the view model.

Fortunately the code to implement this is just a single class:

<script src="https://gist.github.com/jdaigle/a7d8ec3a6867f5250f55.js"></script>

Not only does it select the view based on the view model's class and namespace, but it degrades gracefully in the event that a view model is not supplied, or if the view model does not match any view. It'll fallback to the default logic.

And then on startup, insert the new view engine:

    ViewEngines.Engines.Insert(0,
        new ViewModelSpecifiedViewEngine());

In your controller actions, simply construct the view model and return the view result.

    public ActionResult ListSomeResource() {
         var viewModel
             = new SomeResourceListViewModel();
         ...
         return View(viewModel);
    }

This works well for partial view results too. However there is a hiccup: the use of the `Partial()` HTML helper is sort of broken by since it does not have an override that only accepts a view model object. It *requires a partial view name*... how annoying. But easy enough to work around with this new view engine; the view name can be any string with length > 0. So I simply create an extension method to make it easier:

    public static MvcHtmlString PartialView(
        this HtmlHelper htmlHelper,
        object model)
    {
        // minor hack, since all the internals of
        // finding a partial view require a partialViewName
        return htmlHelper.Partial("null", model);
    }

This is a great technique to adopt with composite or nested views. For example, I have a view model for a page (parent) that contains one or more partial (children) views. There is a list, and for each item I want to render a partial. So in the parent view model I simply have children view model objects. When passed to the `PartialView()` HTML helper, it automatically selects and renders the correct partial view.

The code also works with sub-classed view models. This enables the use of sub-classes to change the behavior of how a view is rendered without implementing complex switch logic either in the view model or the view itself. It's probably rare that one would ever do this, but it's a nice bonus.

Next time I'll talk more about view model conventions and share some tricks that have made development and maintenance smoother and safer.