---
title: 'Improving Async Support in ASP.NET MVC - Part 3 - Async Filter API'
layout: post
---

- [Part 1 - The Current Landscape]({% post_url 2017-03-12-improved-async-aspnet-mvc-part-1 %})
- [Part 2 - Refactoring AsyncControllerActionInvoker]({% post_url 2017-03-13-improved-async-aspnet-mvc-part-2 %})
- [Part 3 - Async Filter API]({% post_url 2017-03-18-improved-async-aspnet-mvc-part-3 %})

The obvious solution is to just copy the API from [ASP.NET MVC Core](https://github.com/aspnet/Mvc/).

    public interface IAsyncAuthorizationFilter : IAuthorizationFilter
    {
        Task OnAuthorizationAsync(AuthorizationContext context);
    }

    public interface IAsyncExceptionFilter : IExceptionFilter
    {
        Task OnExceptionAsync(ExceptionContext context);
    }

    public interface IAsyncActionFilter : IActionFilter
    {
        Task OnActionExecutionAsync(ActionExecutingContext context
            , ActionExecutionDelegate next);
    }

    public delegate Task<ActionExecutedContext> ActionExecutionDelegate();

    public interface IAsyncResultFilter : IResultFilter
    {
        Task OnResultExecutionAsync(ResultExecutingContext context, ResultExecutionDelegate next);
    }

    public delegate Task<ResultExecutedContext> ResultExecutionDelegate();

And that's basically it.

But you'll notice that these async interfaces must also implement their non-async counterparts. Unfortunately MVC5 doesn't have a shared marker interface as exists in MVC Core. And so the framework (and many 3rd party extensions) directly reference the filter types for various reasons. This means that implementations of the async interfaces will just need to implement the non-async methods as a [NOOP](https://en.wikipedia.org/wiki/NOP) - annoying but not the end of the world.

Here's an example implementation based on `FilterAttribute`:

    public class AsyncActionFilterAttribute : FilterAttribute
        , IAsyncActionFilter
        , IAsyncResultFilter {

        public virtual void OnActionExecuting(ActionExecutingContext filterContext) {
        }

        public virtual void OnActionExecuted(ActionExecutedContext filterContext) {
        }

        public virtual async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next) {
            OnActionExecuting(context);
            if (context.Result == null) {
                OnActionExecuted(await next().ConfigureAwait(false));
            }
        }

        public virtual void OnResultExecuting(ResultExecutingContext filterContext) {
        }

        public virtual void OnResultExecuted(ResultExecutedContext filterContext) {
        }

        public virtual async Task OnResultExecutionAsync(ResultExecutingContext context, ResultExecutionDelegate next) {
            OnResultExecuting(context);
            if (!context.Cancel) {
                OnResultExecuted(await next().ConfigureAwait(false));
            }
        }
    }

Next time, putting everything together.