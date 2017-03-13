---
title: 'Improving Async Support in ASP.NET MVC - Part 1'
layout: post
---

## The Current Landscape

ASP.NET MVC ([the non-core version](https://www.nuget.org/packages/microsoft.aspnet.mvc/)) has enjoyed decent support for asynchronous actions. The older Asynchronous Programming Model (APM) approach required inheriting from `AsyncController` and involved creating `ActionAsync()` and `ActionCompleted()` method pairs. The newer Task Parallel Library (TPL), or async/await, approach simplified development significantly by allowing action methods on normal controllers to return `Task<ActionResult>`. On the one hand, the implementation of async inside ASP.NET MVC has been kludgey at best. On the other hand, Microsoft did a good job of tying in async support without any major breaking changes.
 
Despite this, there are a number of use cases that are simply not supported making it *really challenging* to create async-only business libraries.

* Asynchronous filters (action filters, authentication filters, etc.)
* Asynchronous action results (such as rendering Razor views)
* Asynchronous child actions (resulting from lack of async action results)

Granted these problems and more are solved in [ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/). But a lot of world runs on a mix of legacy code, including web forms and old ASP.NET web services. It would have been nice to have some of these features back ported to the ASP.NET MVC 5.x trunk.

**And so I am attempting to just that.** I'm also planning to document the process. Whether or not these changes ever make it into an "official" release, I think this is a worth-while exercise that hopefully someone finds useful.

## The ASP.NET MVC Async Pipeline

Supporting asynchronous action filters is right at the top of my list. All of the code today used to invoke action filters lives in `AsyncControllerActionInvoker`, so I figured it wouldn't be *too hard*. Unfortunately that's where discovered that the MVC pipeline isn't built around async/await, but rather classic APM.

Let's look at how the pipeline is layered today:

    // This is the main entry point after a request is matched to a route
    // (also called when rendering a child action)
    class MvcHandler : IHttpAsyncHandler {
        IAsyncResult BeginProcessRequest(...)
        void EndProcessRequest(IAsyncResult)
    }

    // Called from MvcHandler
    class Controller : IAsyncController {
        IAsyncResult BeginExecute(...)
        void EndExecute(IAsyncResult)
    }
    
    // Called from Controller
    // Invokes MVC filters and executes ActionDescriptor
    class AsyncControllerActionInvoker : IAsyncActionInvoker {
        IAsyncResult BeginInvokeAction(...)
        bool EndInvokeAction(IAsyncResult)
    }

    // Called from AsyncControllerActionInvoker
    // Different concrete implementation based on whether it's a TPL or APM action
    abstract class AsyncActionDescriptor {
        IAsyncResult BeginExecute(...)
        object EndExecute(IAsyncResult)
    }

I want to rewrite `AsyncControllerActionInvoker` so that its internal logic is based on async/await, which will hopefully clean up the code quite a bit. But I suspect that wrapping Task/APM at both the entry point and when calling to `AsyncActionDescriptor` will result in some performance loss. Which means I might as well rewrite the *whole stack*. But then this is a breaking change (like ASP.NET Core).

## Getting Started

The source code today is still hosted on [https://aspnetwebstack.codeplex.com/](CodePlex). I decided to fork the repository and push my work to a branch on GitHub: [https://github.com/jdaigle/aspnetwebstack/tree/async-pipeline-cleanup](https://github.com/jdaigle/aspnetwebstack/tree/async-pipeline-cleanup).

I'm not sure yet if I should modifying the code in `System.Web.Mvc` and recompile, or if I should try implementing the stack in an separate assembly and replace at runtime (introducing breaking changes will obviously have an effect on that).

Next time I hope to have a working build of the revised async pipeline (with all existing unit tests passing). And then I'll take on designing an API for the async filters.

But in the meantime if anyone has any comments or feedback please feel free to leave them below or contact me.