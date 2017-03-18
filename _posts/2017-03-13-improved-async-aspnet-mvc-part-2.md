---
title: 'Improving Async Support in ASP.NET MVC - Part 2 - Refactoring AsyncControllerActionInvoker'
layout: post
---

- [Part 1 - The Current Landscape]({% post_url 2017-03-12-improved-async-aspnet-mvc-part-1 %})
- [Part 2 - Refactoring AsyncControllerActionInvoker]({% post_url 2017-03-13-improved-async-aspnet-mvc-part-2 %})
- [Part 3 - Async Filter API]({% post_url 2017-03-18-improved-async-aspnet-mvc-part-3 %})

[Last time]({% post_url 2017-03-12-improved-async-aspnet-mvc-part-1 %}) I began by discussing several of the shortcomings of the current ASP.NET MVC async implementation. Specifically the lack of async filters (action filters, etc.).

I've decided that the most straight-forward approach is implement a refactored `AsyncControllerActionInvoker`. This is the component within the MVC pipeline responsible for invoking filters.

I've started a new project, [MvcAsync](https://github.com/jdaigle/MvcAsync), for the refactored code.

## Refactoring APM to TAP

As I mentioned, most of the MVC pipeline is written using the classic Asynchronous Programming Model (APM) which requires Begin/End method pairs. This results in very convoluted code that is difficult to reason about.

Fortunately it is possible to wrap APM using the Task-based Asynchronous Pattern (TAP). [Stephen Cleary](https://twitter.com/aSteveCleary) has [written](http://blog.stephencleary.com/2012/07/async-interop-with-iasyncresult.html) and great deal about the topic and has some wonderful [resources]( https://github.com/StephenCleary/AsyncEx).

You can compare [my implementation](https://github.com/jdaigle/MvcAsync/blob/9d1c50f3e3/MvcAsync/AsyncControllerActionInvokerEx.cs) to the [original](https://github.com/jdaigle/aspnetwebstack/blob/v3.2.3/src/System.Web.Mvc/Async/AsyncControllerActionInvoker.cs) to see just how stark the difference is.

And because I wrapped the incoming and outgoing APM, I was largely able to re-used the existing unit tests to assert that my implementation is correct (for now).

## Performance

By wrapping both in the incoming and outgoing APM methods in Tasks, I was originally worried about performance. So of course I decided to create a benchmark (using the fantastic [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) library).

I was pleasantly surprised by the results. Performance is nearly the same, and sometimes *better* in my Task-based implementation.

                                                            Method |       Mean |    StdDev |
    -------------------------------------------------------------- |----------- |---------- |
     AsyncControllerActionInvokerEx_BeginInvokeAction_NormalAction | 17.5357 us | 0.5481 us |
          AsyncControllerActionInvokerEx_InvokeAction_NormalAction | 16.9045 us | 0.2451 us |
       AsyncControllerActionInvoker_BeginInvokeAction_NormalAction | 17.1643 us | 0.1946 us |

Of course I'm assuming there would be an even greater improvement in performance if the entire pipeline could be refactored to use TAP. Unfortunately that would require a lot of changes, many of them breaking changes.

## Using `AsyncControllerActionInvokerEx` at Runtime

Fortunately it's pretty easy to swap in a custom ActionInvoker at runtime. As a developer you have a few options:

* You can create an implementation of `IAsyncActionInvokerFactory` or `IActionInvokerFactory` and register with your DI container.
* You can directly register an `IAsyncActionInvoker` or `IActionInvoker` with your DI container.
* You can simply assign the `ActionInvoker` property on `Controller` in a constructor.

## Next Time

Next time I hope to start planning out the API for new async filters.