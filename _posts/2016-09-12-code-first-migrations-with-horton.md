---
title: '"Code-First" Migrations with Horton'
layout: post
---

Ask any MS SQL Server database expert about Entity Framework, particularly "code-first" migrations, and you'll get responses like this:

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/JosephDaigle">@JosephDaigle</a> &quot;Code First&quot; = Database Last.</p>&mdash; Geoff Hiten (@SQLCraftsman) <a href="https://twitter.com/SQLCraftsman/status/763448503741931520">August 10, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

It's true. The entire concept of "code-first" can be distilled into one simple description; your application code generates your database schema. If you're using EF migrations, then the typical workflow is to run *add-migration* which compares your DbContext's current model with what it thinks is the previous model (based on the last migration applied to the database) and generates a new migration from the difference.

There are two fundamental problems with this "code-first" approach. First, EF generates awful defaults. For example, it'll create an index for each foreign key of a table (whether you need it or not!). It also generates FK constraints with `ON DELETE CASCADE`, something I rarely want in my schema. Developers will almost never change these defaults which can result in sub-optimal schemas.

This directly leads to the other fundamental problem; auto-generated migrations allow developers to further distance themselves from the database. It's easy to blindly trust the schema that EF generates. And ultimately I think this does actual harm by abstracting something (the database schema) which shouldn't be abstracted in the first place!

There a lot of other problems with EF migrations in general (such as poor tooling, lack of idempotency, and the requirement to implement "Down" migrations), but that's content for another article.

### Scaffolding Migrations with Horton

Historically I've frowned upon scaffolding migrations. Despite this, I still often find myself copying/pasting existing migrations and changing names and identifiers - it's so much easier to start from something rather than nothing. Also, teams that use EF "code-first" migrations are accustomed to *always* scaffolding (in fact, there is no other way to add a migration).

And so I think there might be a gap in a lot of SQL-migration-based tooling: a way to scaffold a new migration to reduce time and errors. But the keyword is "scaffold": the migration should always be reviewed and cleaned up before committing.

**To this end, I've added an experimental feature to Horton to scaffold new database migrations.**

The code [lives in a branch](https://github.com/jdaigle/Horton/tree/migration_gen_plugin), with the most important class being the EF [DiffTool](https://github.com/jdaigle/Horton/blob/migration_gen_plugin/src/Horton.MigrationGenerator/EF6/DiffTool.cs).

This feature uses EF's "Metadata Workspace" to analyze the mapping entity model and compare it to a physical database schema. Specifically, the tool looks for new tables and schemas, new or altered columns, and new possible FK constraints.

Why not dropped objects? Well, just because something was removed from the code doesn't mean it should be removed from the database. This I think is another fundamental problem with EF's "code-first" migrations. 

How does it work? Horton has learned a new command named `ADD-MIGRATION`. You simply execute this command, and if any changes are detected then it will prompt for the "name" of the new migration to write. The migration is written to your migration directory and given the next sequence number.

There are a couple of other minor points:

* The command works from a strict set of conventions:
* Your concrete DbContext implementation must have a constructor which accepts a connection string or connection string name.
* The path to the assembly containing the DbContext is defined in a `dbcontext.path` file in the root of your migrations directory. Relative paths are allowed, and encouraged.

Since there is a dependency on EF, I didn't want to add this feature to the core Horton.exe engine. And so I quickly invented a command plugin system. Commands are dynamically added at runtime via reflection. Right now it's really simple: Horton simply scans its app domain directory for .NET assemblies and registers any new command it finds.

A final word: this is an experiment and has not been used for production purposes *yet*. There are still some issues to work out, particularly regarding "legacy" schema and models.
