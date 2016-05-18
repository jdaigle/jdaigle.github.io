---
title: Command Pattern Therapy
layout: post
---

I was planning to write a lengthy article arguing about the merits of the command pattern and how it relates to "clean" web application design. But while researching for this article I rediscovered a series by Jimmy Bogard entitled _Put your controllers on a diet_:

* [https://lostechies.com/jimmybogard/2013/10/10/put-your-controllers-on-a-diet-redux/](https://lostechies.com/jimmybogard/2013/10/10/put-your-controllers-on-a-diet-redux/)
* [https://lostechies.com/jimmybogard/2013/10/22/put-your-controllers-on-a-diet-defactoring/](https://lostechies.com/jimmybogard/2013/10/22/put-your-controllers-on-a-diet-defactoring/)
* [https://lostechies.com/jimmybogard/2013/10/23/put-your-controllers-on-a-diet-a-survey/](https://lostechies.com/jimmybogard/2013/10/23/put-your-controllers-on-a-diet-a-survey/)
* [https://lostechies.com/jimmybogard/2013/10/29/put-your-controllers-on-a-diet-gets-and-queries/](https://lostechies.com/jimmybogard/2013/10/29/put-your-controllers-on-a-diet-gets-and-queries/)
* [https://lostechies.com/jimmybogard/2013/12/19/put-your-controllers-on-a-diet-posts-and-commands/](https://lostechies.com/jimmybogard/2013/12/19/put-your-controllers-on-a-diet-posts-and-commands/)

Take 15 minutes and read them. Seriously, I'll wait. By and large these articles cover pretty much every argument I would make for restructuring your web application and business logic API into sets of commands and queries.

Here's a quick recap of the the most important take-aways from his series:

* The controller is a "lousy" place for business logic. And terrible for unit testing. We want them to be slim and basically only responsible for converting data to and from requests and responses.
* Don't further abstract existing abstractions. I.e., don't bother creating wrappers around your ORM. And *please don't* create an `IRepository<T>` or `IUnitOfWork`.
* GET/POST should conform to semantics. A POST is a "command" and a GET is a "query". Queries should have no side effects (other than logging, etc.) and return data. A command may mutate state, and returns a status code and sometimes a resource identifier.
* Design your business logic API using objects representing "command" and "query" operations. These operations are handled by a mediator with a _single interface_.

**I want to focus on that last point: building a single service interface.** Far too many applications I've seen suffer from a new type of "dependency hell". It's not uncommon to see a Controller with a constructor that has literally _dozens_ of "service" and "repository" interface parameters.

**Interfaces are terrible for designing a constantly evolving businesss API.**

Interfaces have a number of problems. When I'm adding a new operation, should I create a new interface or should I add it to an existing interface? If so, which one?

**Adding a member to an interface is a breaking change.** Granted, in most applications there is only a single implementation (which begs the question, _why do we even need the interface_?). But often we create mock/stub implementations for unit tests. And all of those tests need to be updated even if we're not touching those features!

It's usually quite difficult to implement cross cutting concerns with interface-based services. Such as database transactions (you *are* using explicit database transactions right?) and auditing/logging. There are some neat proxy frameworks out there that generate runtime code which proxy the interface calls and allow you to implement these cross cutting concerns. But it doesn't *feel* right to me.

In my view, the command pattern is ideal as it clearly represents a pipeline:

    HTTP POST → Controller Action → Create Command → Execute → [stuff] → Handler → Return

The POST is handled by a controller action. The controller action is responsible for a) creating a command and b) passing it to the command processing engine. The command processing engine executes a pre-configured pipeline of "behaviors". You can think of these as not unlike ActionFilters in ASP.NET MVC. These can be anything from starting a database transaction/unit of work, to security checks, to logging and auditing. Ultimately the associated command handler method is called, passing the command object as a parameter.

In fact, you could go so far as to allow ASP.NET MVC to create the command object and bind the properties:

    [HttpPost]
    public ActionResult CreateCustomer(CreateCustomerCommand command) {
        command.Validate(); // ensure the command properties are all valid from the POST
        CommandProcessor.Execute(command);
        return OK;
    }
    
How awesome is that? And the best part is that adding new features typically doesn't require modifying existing code. Instead we're just adding new classes.

In fact, we can take it further and get rid of our dependency on a dependency injection framework. But that's an argument best left for another article.

I wouldn't necessarily always design my application this way. It can be overkill for small apps that rarely change. However, **for larger applications that are constantly changing, the command pattern is invaluable.** Software architecture is all about managing dependencies and churn. How easily can you add new features without touching existing code or introducing new dependencies?

I strongly recommend that you consider using a single command/query framework for your business logic services, such as [MediatR](https://github.com/jbogard/MediatR). Or [write your own]({% post_url 2016-05-12-evolving-command-handler-design %}), it's not that hard and can be fun.