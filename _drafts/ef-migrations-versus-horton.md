---
title: Entity Framework Code First Migrations versus Horton
layout: post
---

A lot of .NET developers use Entity Framework these days. EF itself is a decent ORM and data access layer. But it's not without problems: it's very slow compared to alternatives and quite a memory hog. But most developers are comfortable using EF, and it makes them more productive which is a *good thing*.

Within the Entity Framework nomenclature is this idea of **code first** versus **database first** (there's also a **model first** concept which I think has been lost to time). With "database first", the idea is that you'll generate EF entities based on an existing database schema. When the schema changes, you re-generate the code. But developers tend to have an aversion to generated code, whether for legitmate reasons or not. And so EF also ships with a "code first" approach in which you manually write entity classes which will map to a database schema - this is exactly the same approach that other ORM frameworks such as NHibernate adopt.

### Entity Framework Code First Migrations

The concept of "code first" in Entity Framework often extends beyond simply hand writing entities and mappings. It usually includes *generating your database schema from your entity model*! This is accomplished using **Entity Framework Code First Migrations**. For sanity, I'll simply refer to it as **EF Migrations** from here on.

EF Migrations merge two concepts. It takes the concept of "migrations" as a database versioning strategy and couples it to Entity Framework. Using the EF toolchain, you start by "adding a migration". If it's the first migration, EF takes the entity model represented by your code and generates a code file that will generate the necessary DDL to create the schema represented by that entity model.

That's a mouthful: but in essence a EF Migration is an C# class with it's own domain specific language (DSL) that at runtime generates SQL Server specific data definition language (DDL) commands. But an EF migration *also serializes the state of the entity model when that migration was created*. The next time you "add a migration", EF compares your code against the serialized state of the migrations and generates a new migration class with the differences.

To run your migrations, EF operates in one of two modes. The first, and default, mode is to run the migrations automatically on application startup. The second mode is to explicitly run the migrations with EF's `migrate.exe` tool.

Conclusion: **EF Migrations are kind of amazing. And I think they should be avoided!**

### The False Choice

I think the idea of "code first" versus "database first" is a false choice. **For any long-lived application you must consider your database schema as independent from your application code.** Why? Because *your database will outlast your application code*. Applications are constantly rewritten or replaced. The frameworks and platforms we use to build these applications change. But the database changes at a much slower pace. That's because the *data* in your *database* is usually the most important assest of your company. Yes, the application is extremely important and may even be your "product". But it can be rewritten or replaced. Your data likely cannot.

### Principal #1: We should be designing our database schema correctly, and independent of our application code and frameworks.

One of the biggest failures of the model state based migrations is the complexity that it adds when working in teams or branches. People have [run into problems](http://stackoverflow.com/a/19158039) with this all time. Just look how complex [Microsoft's own guidance](https://msdn.microsoft.com/en-us/data/dn481501.aspx) is on the topic! The simple fact that one must sometimes create "dummy" migrations" is just ridiculous.

### Principal #2: Database migrations should not be tied to any particular application framework or tool.

What if we stop using Entity Framework in the future? Or worse, what if a future version of Entity Framework is incompatible with our existing migrations? I think it's inevitable that any existing EF Migrations will be rendered broken at some point in the future.

However, if we wrote our migrations simple as a set of SQL scripts that conform to some naming convention, they will easily remain compatible with virtually any migration tool we adopt.

**There is almost no value to writing migrations in a DSL other than SQL.** The most obvious value comes from being database agnostic - we can write migrations that target different SQL dialects. However in practice this rarely works out well. It's almost always better to create explicit migrations in the specific dialects targeted.