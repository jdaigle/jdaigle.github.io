---
title: Aftermarket ASP.NET MVC - Part 5 - Feature Folders
layout: post
---

This is **part 5** of a multi-part series on "fixing" some of the inherit design problems with ASP.NET MVC.

- [Part 1 - Introduction]({% post_url 2016-01-05-bridging-aspnet-mvc-part-1 %})
- [Part 2 - The View Engine]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %})
- [Part 3 - View Model Conventions]({% post_url 2016-01-07-bridging-aspnet-mvc-part-3-view-model-conventions %})
- [Part 4 - Routing and URL Generation]({% post_url 2016-01-13-bridging-aspnet-mvc-part-4-routing-and-urls %})
- [Part 5 - Feature Folders]({% post_url 2016-02-19-aftermarket-aspnet-mvc-part-5-feature-folders %})
- Part 6 - ActionResults and Content Negotiation

I've also setup a [repository on GitHub](https://github.com/jdaigle/aspnetmvc5demo) that includes many of these experiments and implementations.

Expanding on the theme from last time:

> Don't let your routing solution dictate how to organize your code.

ASP.NET MVC establishes the convention of putting all of your *controllers* inside of a `Controllers\` directory and all of your *views* instead of a `Views\` directory. This is, no surprise, extremely similar to the convention established by [Rails](http://rubyonrails.org/). Remember that Rails was the new-hotness when ASP.NET MVC was first developed around 2007-2008.

A prototypical ASP.NET MVC source code layout: 

    MyApp/
    ├── assets/
    │   ├── site.css
    ├── Controllers/
    │   ├── HomeController.cs
    │   ├── ProductsController.cs
    ├── Views/
    │   ├── Home/
    │   │   ├── HomePage.cshtml
    │   │   ├── AboutPage.cshtml
    │   ├── Products/
    │   │   ├── Index.cshtml
    │   │   ├── Detail.cshtml
    │   ├── Shared/
    │   │   ├── _Layout.cshtml
    │   ├── _ViewStart.cshtml
    │   ├── Web.config
    
Yuck!

A lot of people have written about how this is just a terrible way to organize your code. At the top level, you don't see *your application*. You see an MVC framework details leaking through.

I *should* be able to organization my code based on *feature* instead of what type of class it is.

    MyApp/
    ├── assets/
    │   ├── site.css
    ├── Home/
    │   ├── HomeController.cs
    │   ├── HomePage.cshtml
    │   ├── HomePageViewModel.cs
    │   ├── About.cshtml
    │   ├── AboutViewModel.cs
    ├── Products/
    │   ├── ProductsController.cs
    │   ├── Index.cshtml
    │   ├── IndexViewModel.cs
    │   ├── Detail.cshtml
    │   ├── DetailViewModel.cs
    ├── SharedViews/
    │   ├── _Layout.cshtml
    ├── _ViewStart.cshtml
    ├── Web.config

Now you can look at the project structure and get a sense of what the app is! Additionally, all of the code related to a feature is in close proximity. In my opinion this makes development easier, possibly even faster.

### A Quick Word About *Areas*

In theory, the *Areas* feature can get you pretty close to this model out of the box. However I'm not a fan of *Areas*. They were originally designed to work sort of like "sub-projects". In theory they can physically live in their own assemblies, but this causes problems. But at the end of the day *Areas* still suffer, though internally, from the same organizational problems.

I prefer to avoid areas.

### Making Feature Folders Work

Unfortunately *Feature Folders* won't necessarily work out-of-the-box. There are a few things we need to tweak.

First, as a mentioned in [a previous post]({% post_url 2016-01-13-bridging-aspnet-mvc-part-4-routing-and-urls %}), assigning routes to individual actions is important.

Second, the default view engine only knows to look for *views* inside of `Views\`. But we can subclass the default engine using [the technique I wrote about here]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %}) to make it work.

Inside the traditional `Views\` directory lives a special `Web.config`. This does two things: 1) it makes the Razor tooling work and 2) it prevents access to the cshtml files in the directory tree as "web pages" (as the views are commonly deployed, and then compiled at runtime). We may move *some* of the config to root the `Web.config` of our project.

First, we need to register and include the `RazorWebSectionGroup`.

    <configSections>
        <sectionGroup name="system.web.webPages.razor"
                      type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, 
                            System.Web.WebPages.Razor, Version=3.0.0.0,
                            Culture=neutral, PublicKeyToken=31BF3856AD364E35">
            <section name="host" 
                     type="System.Web.WebPages.Razor.Configuration.HostSection,
                           System.Web.WebPages.Razor, Version=3.0.0.0,
                           Culture=neutral, PublicKeyToken=31BF3856AD364E35"
                     requirePermission="false" />
            <section name="pages"
                     type="System.Web.WebPages.Razor.Configuration.RazorPagesSection,
                           System.Web.WebPages.Razor, Version=3.0.0.0,
                           Culture=neutral, PublicKeyToken=31BF3856AD364E35"
                     requirePermission="false" />
        </sectionGroup>
    </configSections>
    
    <system.web.webPages.razor>
        <host factoryType="System.Web.Mvc.MvcWebRazorHostFactory,
                           System.Web.Mvc, Version=5.2.3.0,
                           Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
        <pages pageBaseType="System.Web.Mvc.WebViewPage">
        <namespaces>
            <add namespace="System.Web.Mvc" />
            <add namespace="System.Web.Mvc.Ajax" />
            <add namespace="System.Web.Mvc.Html" />
            <add namespace="System.Web.Routing" />
        </namespaces>
        </pages>
    </system.web.webPages.razor>
    
This essentially makes the Razor tooling and compilation work.

Next, under `<handlers>` in `<system.webServer>` we need to add:

    <add name="BlockViewHandler" 
         path="*.cshtml" 
         verb="*" 
         preCondition="integratedMode" 
         type="System.Web.HttpNotFoundHandler" />

In the default `Views\Web.config`, this handler is registered with the `path="*"`. But we don't want to block access to *all files*, just our cshtml files. Finally we can add an `appSetting` to turn off "webpages":

    <add key="webpages:Enabled" value="false" />

Unless you want normal web pages to work... and why would you at this point? In which case you'll need to leave them enabled and perhaps tweak the handler to not exclude the pages you want.

Alternatively you can copy the default `Views\Web.config` into each of your feature folders. But that's ugly and possibly un-maintainable.

Another interesting problem man run into: if you have a folder named `Foo\`, and a URL such as `~\Foo\` that matches that directory name, by default `System.Web.Routing` won't match that URL because it's a physical folder on disk. In order to force `System.Web.Routing` to match that URL, we need to set `RouteTable.Routes.RouteExistingFiles = true;`. This setting changes the behavior so that it *does not* first check that a file exists before attempting to find a route.

I also encourage setting up routes that explicitly ignore your static assets (CSS/JS, images, HTML files, etc.) to help with performance.

### Things Might Get Better with ASP.NET Core

*Feature Folders almost just work.* I think some of the decisions made in ASP.NET MVC are partially due to adopting the conventions of Rails-like frameworks, but also partly to workaround the inherit properties of ASP.NET.

In ASP.NET, traditionally, you deploy everything as a file. You're static content and your dynamic content (ASPX pages) are mixed. There's no routing, and URLs are reflective of the physical layout of files on disk.

But in almost any modern web application, except for static content, *URLs are an abstraction* over dynamic content.

Luckily ASP.NET Core throws away *everything* and starts from *scratch*. We may, for instance, have a project structure which looks like this:

    MyApp/
    ├── wwwroot/
    │   ├── assets/
    │   │   ├── site.css
    ├── Home/
    │   ├── HomeController.cs
    │   ├── HomePage.cshtml
    │   ├── HomePageViewModel.cs
    │   ├── About.cshtml
    │   ├── AboutViewModel.cs
    ├── Products/
    │   ├── ProductsController.cs
    │   ├── Index.cshtml
    │   ├── IndexViewModel.cs
    │   ├── Detail.cshtml
    │   ├── DetailViewModel.cs
    ├── SharedViews/
    │   ├── _Layout.cshtml
    ├── _ViewStart.cshtml

As you can see, they've introduced a `wwwroot` directory. This is a special folder which contains all of the static content that will be deployed too the root of the web application on the server. Everything else in the project *is just code*.

Fun fact: while you may compile this before deployment, it should be noted that ASP.NET Core has a new build model where it will compile your code at runtime *and* detect changes recompile and relaunch.