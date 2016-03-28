---
title: The Database Versioning Showdown
layout: post
---

Versioning of your database schema is an important component of mature software development. Depending on your database system there are number of tools available to version your schema. While most are adequate for the job of version control, many are not designed for *deployment* of schema changes.

A lot of development shops are practicing some form of continuous deployment (CD). Particularly those who are building software-as-a-service (SaaS). The ability to go from source to a working application, including the database, is critical. 

Focusing on relational databases, most tools fall into one of two camps: 1) model based or 2) migration based.

### Model Based Version Control

What I'm calling **model based** version control refers to maintaining each object in the database (table, view, stored procedure, etc.) as a separate file in your version control system. Most often it takes the form of a SQL script containing the necessary `CREATE` commands for the corresponding database object.

It's easy to see how this approach provides excellent visibility into changes that are made to the schema. You can also see line-by-line source control annotations.

When it comes time to deploy, either upgrading an existing database instance or setting up a brand new one, the model represents a **desired state**. Generally some tool is responsible for *comparing* the model versus existing state of a target database and generating an upgrade script. Some tools do a better job than others when it comes to determining the most efficient and *correct* script to get to the desired state. But regardless, you're relying on some algorithm to figure it out.

The biggest danger desired state upgrades is the potential for data loss. A schema comparison tool will do it's best to determine how to alter the schema correctly without data loss. For example, renaming a column rather than dropping and adding. It's bound to make a mistake.

Perhaps the two most popular tools, especially in the .NET community are Redgate's [SQL Source Control](http:/www.red-gate.com/products/sql-development/sql-source-control/) and Visual Studio [Database Projects](https://msdn.microsoft.com/en-us/library/hh272702(v=vs.103).aspx) using SQL Server Data Tools (SSDT).

### Migration Based Version Control

Rather that representing the database as a model in your version control system, **migrations** are a set of sequentially executable scripts that iteratively alter a database's schema. Often just SQL scripts, and using a naming convention that may include a numeric prefix for ordering, migration scripts are usually very simple to reason about.

They're usually written by hand to execute precise changes in a precise order. As a result there's little additional work required to ensure data integrity during and upgrade.

Migrations are also efficient. Instead of comparing and generating a change script, we simply execute the migrations that haven't been run yet in the correct order. Often a special database table is used to maintain information about the migration scripts that have been executed.

I *think* database migrations first become really popular as part of the ActiveRecord implementation in Rails (i.e. Ruby on Rails) as far back as 2005-2006. You wrote "up" and "down" migrations in Ruby, and a tool that shipped with Rails would upgrade/downgrade the database accordingly. When building later versions Entity Framework, Microsoft adopted a similar pattern for database migrations as part of their "code-first" database strategy. In a future article I'll talk about why you don't need another domain specific language (DSL) on top of SQL DDL, and why it might actually be harmful.

### Why not both?

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">wanted: DB migrations for schema, DB projects for sprocs/functions</p>&mdash; Jimmy Bogard (@jbogard) <a href="https://twitter.com/jbogard/status/710188482279288832">March 16, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Clearly migrations are better suited for schema changes in which data preservation is critical. They naturally map to the how a DBA or developer may make changes: a set of iterative commands performed step-by-step.

However some database objects, such as stored procedures, views, and functions, *can* exist independent of the underlying schema. They're also not really tied to data per se. Instead they're more of an abstraction over the relational model. Stored procedures and functions, in particular, are almost "code" by their nature. As such it does make more sense to treat these like classes that are modified over time.

Ideally we have a tool that mixes both migrations and model/desired state. Fortunately a number of tools exist. I wrote such a tool almost seven years ago (!!) which I know call [Horton (https://github.com/jdaigle/Horton)](https://github.com/jdaigle/Horton). While researching for this article I stumbled across several other .NET based tools. One in particular that stoop out is [RoundHousE (https://github.com/chucknorris/roundhouse)](https://github.com/chucknorris/roundhouse). It's *not quite as old as Horton*, but it's been around long enough and clearly has a well defined community.

I suggest checking these out and figuring out how they might work in your continuous deployment strategy.


### Further Reading

* [Database versioning best practices](http://enterprisecraftsmanship.com/2015/08/10/database-versioning-best-practices/)
* [State vs migration-based database delivery](http://enterprisecraftsmanship.com/2015/08/18/state-vs-migration-driven-database-delivery/)