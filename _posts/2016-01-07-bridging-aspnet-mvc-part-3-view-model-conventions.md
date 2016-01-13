---
title: Bridging ASP.NET MVC - Part 3 - View Model Conventions
layout: post
date: 2016-01-07
---

This is **part 3** of a multi-part series on "fixing" some of the inherit design problems with ASP.NET MVC.

- [Part 1 - Introduction]({% post_url 2016-01-05-bridging-aspnet-mvc-part-1 %})
- [Part 2 - The View Engine]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %})
- [Part 3 - View Model Conventions]({% post_url 2016-01-07-bridging-aspnet-mvc-part-3-view-model-conventions %})
- [Part 4 - Routing and URL Generation]({% post_url 2016-01-13-bridging-aspnet-mvc-part-4-routing-and-urls %})
- Part 5 - The Controller Factory
- Part 6 - ActionResults and Content Negotiation

### 1) Each View Should Have a Strongly Typed View Model

By default, views in ASP.NET MVC have a `dynamic` view model. The underlying object is simply a dynamic key/value bag. While this aids with "rapid development", in my opinion it hurts long term maintenance. You lose static code analysis, compilation time error checking, refactoring support, and some debugging support.

Instead, I think **it's far better to always have a strongly typed view model** for each and every view.

![](/images/strongly-typed-views.PNG)

I also like the convention of **putting view models alongside the view itself**. By convention the class name is the name of the view followed by "ViewModel". Thus your view model's name follows a pattern of `[Project].Views.[Feature].[ViewName]ViewModel`.

### 2) A View Should Only Reference Properties and Methods From the View Model

View engines in ASP.NET MVC, such as Razor, typically compile to classes that inherit from `System.Web.Mvc.WebViewPage`. Unfortunately this includes a lot of cruft that carries over from classic ASP and ASP.NET web forms that inherit from `System.Web.UI.Page`. Essentially, the view has references to so much more
that it probably needs to just simply render a template!

Accessing anything outside of the view model is an anti-pattern. Global or static state is also not a good idea. **Remember that the only job of the view engine is to create a string from a template.** It can be reduced to just a named function that accepts a model a returns a string.

* Don't access HttpContext/Request/Response.
* Don't access the referenced controller.
* Don't access route data.
* Don't query a database or call external services.

***"But I need to know if a user is authenticated to show/hide things."*** Wrong. The view is responsible for rendering content and controls. If content or a control should be conditionally rendered, then the view model should include state that explicitly declares that condition.

### 3) URLs for Links and Buttons Should Be Part of the View Model

ASP.NET MVC has the ability the generate URLs for routes based on a controller/action pair. This is great since since you don't need to hardcode URLs all over the place (except you still deal with string constants). **But you should not use `HtmlHelper` or `UrlHelper` to create a link/button or generate an action URL from within the view template.** Instead, any generated URLs should be properties in the view model. This helps to further keep the view's template clean and without references to constants, or worse, controllers and actions.

### Additional Tip: Pass a Razor Template as a Parameter to a View Model Helper Function

In your view model you can create methods that accept Razor templates as `Func<object, HelperResult>`.

    public MvcHtmlString RenderIfCanEdit(
        Func<object, HelperResult> template)
    {
        if (this.CanEdit) {
            return MvcHtmlString
                .Create(template.Invoke(null)
                .ToHtmlString());
        } else {
            return MvcHtmlString.Empty;
        }
    }
    
Now in the view, you can do awesomeness like this:

    <div>
        @Model.RenderIfCanEdit(
        @<p>
            ...
        </p>
        )
    </div>

Depending on the scenario this can be a lot cleaner than having nasty looking if-statements. Here's an [article by Phil Haack](http://haacked.com/archive/2011/02/27/templated-razor-delegates.aspx/) that explains it more detail.

### Further Reading

* [C# Razor Syntax Quick Reference](http://haacked.com/archive/2011/01/06/razor-syntax-quick-reference.aspx/)
* [Ten Tricks for Razor Views](http://odetocode.com/blogs/scott/archive/2013/01/09/ten-tricks-for-razor-views.aspx)