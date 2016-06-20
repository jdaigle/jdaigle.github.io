---
title: Building a Modern Enterprise Business Logic Layer 
layout: post
---

This is a continuation of my previous article entitled "Command Pattern Therapy". A lot of the content of this article is inspired by (and borrowed from) Jimmy Bogard's talk [SOLID Architecture in Slices not Layers](https://lostechies.com/jimmybogard/2015/07/02/ndc-talk-on-solid-in-slices-not-layers-video-online/) - I highly recommend taking the time to watch it.

### The Old Way

If we think about the typical way we, as developers, are often taught to design systems, it's as a series of layers. In theory, each building upon the next.

    [ UI Layer             ]
    [ Business Logic Layer ]
    [ Data Access Layer    ]
    [ Database             ]

Each layer is designed to fully encapsulate the next. Communication between the layers is achieved through interfaces. For example, our `data access layer` (DAL) exposes "repositories" and perhaps a "unit of work". The `business logic layer` (BLL) encapsulated the DAL into "services". Our `UI layer` might simply be an ASP.NET MVC application.

*Aside*: in the "old days", the DAL would often encapsulate low-level ADO.NET usage (inline SQL, data tables/adapters, generated code, etc.). This actually made sense, because ADO.NET is not a good application API. However, now we have powerful ORM frameworks such as NHibernate and Entity Framework. These frameworks provide all of the features, and more, that we would often build as the DAL. And yet *we still* try to encapsulate these frameworks! It's just silly and has to stop.

When using layers our application code is often organized by layer. To build a feature, we must implement it at each layer. For example, I'll create a `Person` database entity, a `PersonRepository`, a `PersonService`, and finally a `PersonController`. Most of the time our model is really just dumb CRUD based stuff: reading an entity or set of entities, adding, updating, removing. There's very little actual "business logic" except for some basic validation or constraints. And yet, we have all of these "layers" and components which must be wired up. Take any of these components for a CRUD model, and you'll probably see only 1 or 2 lines of code that simply delegate to the next layer down!. Where does the actual logic live?!

### Breaking Down SOLID

Jimmy has a fanstatic slide which shows a stereotypical "service" interface from one of these types of systems. It sort of looks like this:

    public interface IPersonRepository : IRepository<Person> {
        Person GetByPersonId(string personId);
        Person GetByPersonldForFoodProcessing(string personld);
        Person GetByPersonldForCoffeeProcessing(string personld);
        Person GetByPersonldForSpendMoreGetMoreProcessing(string personld);
        Person GetByPersonIdWithNoEagerFetch(string personld);
        Person GetByPersonldForBirthdayRewardProcessing(string personld);
        
        Person[] GetByPersonldOrEmail(string personld, string email);
        
        IEnumerable<PersonWithThermbarUpdates> GetUpdatedPeople();
        void RemoveUpdatedFlagFromAllPeople();
        void RenamePerson(string personId, string firstName, string lastName);
        Person FindByEmail(string email);
        Person[] GetPeopleWithPersonldlnATempTable(string tempTableName);
        Person GetByldFetchSpendEntries(Guid id);
        Person GetByldFetchBenefits(Guid id);
        PersonWithExpiredFoodEntry[] GetPeopleWithExpiredFoodEntriesln(DateTime date);
        
        ... etc  ..

What do these methods have anything to do with each other? At first glace, it might seem like they all have to do with the `Person` entity, and operations there of. And while that's not *wrong*, you are unlikely to ever need more than 1 or 2 of these methods for any part of the UI!

As a result, two very important [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles are violated: the [interface segregation principle](https://en.wikipedia.org/wiki/Interface_segregation_principle) and the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). I won't go into too much more detail because I hope that after skimming the provided links it comes obvious.

**What I have come to realize over time is that interfaces are *the wrong* abstraction for building a constantly changing business logic API.**

### Web-First Design

I should note that much of this article is written with the understanding that these days we are predominantly building web applications and web APIs. The patterns described below may, or may not, be the right approach for a traditional desktop application or a long-running service.

Let's think about how users interact with our web applications. They're using a web browser (or perhaps some application that's consuming our app as an HTTP API). All web apps can be broken down into two general interactions:

1. The HTTP "GET". Semantically a GET should be idempotent and side-effect free. It's used to *query* for something (hence a *query string*).
2. The HTTP "POST". (For simplicity, I'm lumping all of the other rarely-used verbs together). A post is *not side-effect free* and *not idempotent*. A user issues a POST request when intention is to alter state in some way. The user is *command*ing the system to do something. Parameters for the *command* are often provided as key/value pairs that make up the HTTP request body. The response to the POST can vary from a simple 2xx (success), a 4xx (something wrong with the request), or a 5xx (something bad happened on the server). There may or may not be body. Basically, commands just tell you whether or not they were successful.

*Aside:*, yes I know that sometimes we use a POST to issue a query with more data than fit into a query string. And I know that sometimes we do horrible things with a GET (like logging out/etc). But I think my point is still valid.

So that's it; we have **queries** and we have **commands**. This is the essence of **Command/Query Responsibility Segregration** (CQRS). We are going to have separate models for commands and queries.

### Breaking up the Interfaces and Collapsing the Layers

Let's look again at that stereotypical service interface I printed above. What do we see? The methods on the interface can pretty clearly be partitioned into methods that *query* the database, and methods intend to update the database (i.e. a *command*).
