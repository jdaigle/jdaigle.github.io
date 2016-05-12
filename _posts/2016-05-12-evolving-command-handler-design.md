---
title: Evolving Command Handler Design
layout: post
---

It's no surprise that I'm a big fan of the command and mediator patterns when building business logic and service APIs. Particularly when compared to the traditional approach to abstracting services: C# interfaces with methods.

It's awesome to be able to reduce *all* of your service interfaces down to just one:

    public interface IServiceAPI
    {
        TResult Execute<TCommand>(TCommand command);
    }

But this article is not about the merits of the pattern, or how to design your API layer. This article is going focus on my evolving ideas around implementing the actual command "handler" logic.

### In the beginning

My very first technique is borrowed directly from NServiceBus when I first introduced to it seven years ago.

    public interface ICommandHandler<TCommand, TResult>
    {
        TResult Execute(TCommand command);
    }

    public class MyCommandHandler : ICommandHander<MyCommand, MyCommandResult>
    {
        public MyCommandHandler(DbContext ... etc)
        {
            // ... set instance variables
        }
        
        public MyCommandResult Execute(MyCommand command)
        {
            // ... do stuff, access instance variables/etc.
        }
    }

**It's very simple**: each command is handled by a class that implements `ICommandHandler<TCommand>`. A particular class *can* implement multiple command handlers, however in practice Single Responsibility Principal (SRP) kicks in and you would typically create a separate class per command.

Dependency injection in very straightforward too. Dependencies are supplied to the class' constructor at runtime; often the command handler itself is constructed and managed by some IoC framework.

During application startup the bootstrapping system will scan for non-abstract types that implement the command handler interface and register them against the command types. At runtime, the class is dynamically created and the method invoked. It's no surprise that this approach is quite similar to how ASP.NET MVC controllers and actions work. In fact I've borrowed [techniques from its code](https://aspnetwebstack.codeplex.com/SourceControl/latest#src/System.Web.Mvc/ActionMethodDispatcher.cs) in my own frameworks.

So while the code is simple to understand, **it is a lot of ceremony and boilerplate**.

### Enter functions

Consider this: *an interface that defines only a single method is not an interface; it is a delegate*. Why do we need to instantiate a class just to call our command handler function? What if we designed it just as a static method?

    public static MyCommandResult Execute(MyCommand command)
    {
        // do stuff
    }

We can simply declare, somewhere, a static method that matches our "interface", or delegate: `public delegate TResult Execute<TCommand>(command);`. Now it's trivial to wire up code that executes appropriate delegate when a command of type `TCommand` is supplied.

That's great, but what about my dependencies? Functional programming has solved this with [currying](https://en.wikipedia.org/wiki/Currying).

    public static MyCommandResult Execute(MyCommand command, IDependency foo, IOtherDependency bar)
    {
        // do stuff
    }

The idea is that our framework will, at runtime, compile a new expression that calls our command handler method with resolved dependencies. That compiled expression will match the common delegate. Again, this is nothing new - ASP.NET MVC does this with model binding of action parameters. And we have one fewer allocation since we're not creating an instance of a class just to call a method.

As far as automatically registering command handlers, there are a few techniques I've used the past. But the simplest has been to decorate each command handler method with an attribute, such as `[CommandHandlerAttribute]`. This makes it super easy, at runtime, to scan and find the matching methods.

I've experimented with static registration techniques, but they're not pretty. I think this is mostly due to limitations in the expressiveness of C#. You can find an [example here](https://gist.github.com/jdaigle/aa10e138e2802b3c42c09e3c906c0fdc).

**Now we have lightweight static methods that are dynamically invoked.** But where do we put them? Unfortunately for .NET, all methods including static methods must be declared on a class. Do we create one static class per handler? Do we put them all in one giant class? How do we group them? How should we name the static class? I suppose this is a really minor concern, but it annoys me because naming and organization can be difficult.

### The next generation

While thinking about the "problem" of organizing my command handlers, I was recently made a connection to how frameworks like [NancyFx](http://nancyfx.org/) define modules. If you look at a typical [Nancy module](https://github.com/NancyFx/Nancy/wiki/Exploring-the-nancy-module) you might notice that, in a way, it's just defining and mapping command handlers (pedantically speaking they're "HTTP request" handlers, but conceptually the same).

What if instead of a "NancyModule" we could build a "CommandModule"?

*Caveat: this is all experimental. I've not tried this technique in production code.*

    public class ProductModule : CommandModule
    {
        public ProductModule(IFoo foo, IBar bar)
        {
            Handle<AddProductModel>(command =>
            {
                foo.Something();
                return 1;
            });

            Handle<AddProductReview>(command =>
            {
                foo.Something();
                bar.SomethingElse();
            });
        }
    }

The way this, like NancyFx, works: on startup "modules" are discovered and initially constructed. This first pass results in a registry that maps the "request" (command type in my case) to a particular module and lambda. At runtime, when a "request" (i.e. command) is executed, the appropriate module is constructed and the corresponding lambda is executed.

There are some interesting properties about this design:

* The command handler lambda can compile with a closure around any referenced dependencies, which are passed via the module constructor.
* When the command is executed, we do instantiate an instance of the module (one per "request").
* If the lambda was compiled with a closure, then another class is instantiated (one per "request") with the captured references.

That is extra overhead, but maybe it's a good tradeoff in order to have more expressive code? This seems like the perfect way to organize this code. Each module can be built with a clear and single purpose. I can use existing IoC frameworks to handle module construction and dependency injection. The framework itself becomes a lot simpler.

Some brainstorming: Maybe there is a way to compile an expression which skips the module construction at runtime. We know about the about the `MethodInfo` of the lambda. So we know about the anonymous `DeclaringType`. If the `DeclaringType` is for a closure, we can inspect it's fields. At runtime, we can construct that object and populate the fields from an IoC container. Then use that call the method. So that's one fewer allocation.

### Wrapping up

Over the years my ideas and design for command handlers has evolved.

1. First were full-blown classes implementing a interface.
1. Then I reduced those down to simple static methods that live *somewhere*.
1. Finally, I've experimented with "modules" that declare command handlers as lambdas.

Each technique has pros and cons. Different applications, or contexts within a single application, may be suited to one technique over another. Some teams might be comfortable with one technique over another. But I'm curious about you, dear reader, and what technique you like and why? 