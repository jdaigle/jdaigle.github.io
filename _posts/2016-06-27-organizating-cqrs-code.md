---
title: Code Organization, CQRS Style
layout: post
---

I am a [huge believer]({% post_url 2016-05-17-command-pattern-therapy %}) in the Command/Query pattern for my applications' internal services and APIs, particularly for those services that touch a database.

But I've been constantly challenged with code organization. What is the right approach for a particular project and team?

I think you might find a lot of .NET developers, especially those that use ASP.NET MVC, lean towards code organization *by function*. What does this mean?

So in ASP.NET MVC, we have separate folders (and therefore namespaces) for *Controllers*, *Views*, and pretty much anything else like our *Model*, *ActionFilters*, *Helpers*, etc. This kind of insanity often manifests itself in other parts of the code base. Therefore we may end up with something like this:

    MyServiceProject/
    ├── Commands/
    │   ├── SomeFeature/
    │   │   ├── FeatureUseCaseCommand.cs
    ├── CommandHandlers/
    │   ├── SomeFeature/
    │   │   ├── FeatureUseCaseCommandHandler.cs
	├── Queries/
    │   ├── SomeFeature/
    │   │   ├── FeatureUseCaseQuery.cs
    ├── QueryHandlers/
    │   ├── SomeFeature/
    │   │   ├── FeatureUseCaseQueryHandler.cs
    │ ... etc...

Obviously I would prefer to invert the hierarchy and organize the code by feature. For example:

    MyServiceProject/
    ├── SomeFeature/
    │   ├── Commands/
    │   │   ├── FeatureUseCaseCommand.cs
    │   ├── CommandHandlers/
    │   │   ├── FeatureUseCaseCommandHandler.cs
    │   ├── Queries/
    │   │   ├── FeatureUseCaseQuery.cs
    │   ├── QueryHandlers/
    │   │   ├── FeatureUseCaseQueryHandler.cs
    │ ... etc...

That's a bit better. I've also tried flattening the hierarchy:

    MyServiceProject/
    ├── SomeFeature/
    │   ├── FeatureUseCaseCommand.cs
    │   ├── FeatureUseCaseCommandHandler.cs
    │   ├── FeatureUseCaseQuery.cs
    │   ├── FeatureUseCaseQueryHandler.cs
    │ ... etc...

While this has less ceremony, the code itself can really start to feel cluttered.

But one day I had a thought: why does the *Command* and *CommandHandler* need to be in separate files? They're likely to change together, so it's not a violation of the Single Responsibility Principle. and you can easily define two classes in the same file. So now we have this:

    MyServiceProject/
    ├── SomeFeature/
    │   ├── FeatureUseCaseCommand.cs
    │   ├── FeatureUseCaseQuery.cs
    │ ... etc...

That's pretty clean. I'm also experimenting with nested types:

    public static class FeatureUseCase {

        public sealed class Command : ICommand {
            public string Data { get; set; }
        }

        public sealed class Handler : ICommandHandler<Command> {
            public void Handle(Command command) {
                // handle command
            }
        } 
    }

This, I think, is very cool. Instead of having *FooCommand* and *FooCommandHandler*, I simply have a class named *Foo* with *Command* and *Handler* as nested types. Functionally it's no different from the former design. But maybe this reads better. Additionally, if I later decide to add something like validation, I can simply add whatever classes/code I need as a new nested types. The related code lives together in a single code file, and my project is clean.