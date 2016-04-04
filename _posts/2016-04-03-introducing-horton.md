---
title: Introducing Horton
layout: post
---

[![Horton](https://upload.wikimedia.org/wikipedia/en/d/d5/Horton_the_Elephant.jpg)](https://en.wikipedia.org/wiki/File:Horton_the_Elephant.jpg)

### What is Horton?

Horton is the simple database migration utility. It's a little program that does one thing: enables versioning of database schema through SQL based migration scripts.

You'll find a "**getting started guide**" in the project's [GitHub repository](https://github.com/jdaigle/Horton). Horton is also released through GitHub. You can find and download the latest release from the [releases tab](https://github.com/jdaigle/Horton/releases).

### What does it do?

**It may help to read this first**: I recently [wrote about database versioning with migrations]({% post_url 2016-04-02-database-versioning-showdown %}).

Horton handles migrations for schema  objects *and desired state* for "code" like objects (stored procedures, views, etc.). It was written to help apply database change scripts to production servers in a fully predictable and automated fashion (i.e. continuous deployment). It's grown a number of features in recent years, but has never strayed from its very focused purpose. 

### Why am I "Introducing" it now at version 4.0?

Horton is actually the latest incarnation of a SQL migration tool I wrote almost seven years ago. The first commit was in September 2009! And even though it was *published* on GitHub, it was always just an internal tool and I never treated it as a real open source project. But I'm trying to change that.

Versions 1-3 did not strictly follow [semantic versioning](http://semver.org/) either. There were few, if any, breaking changes over its lifetime.

For version 4, I decided to rewrite it.

- I rewrote the command line parser. Actually, it uses [NDesk.Options](http://www.ndesk.org/Options) which is the best .NET command line parser I know of. The command line arguments are similar, but different. Notably you can execute different COMMANDs from the same tool. 
- I redesigned the `schema_info` table. This is biggest breaking change as it renders the tool entirely incompatible with previous version. There's an open GitHub issue suggesting how solve this if it becomes a problem (though highly unlikely because not many projects used the old tool!).
- There's cleaner separation between database specific code and the general horton program code. Data access is all based on ADO.NET, so in theory I should be able to support any provider.

I'd love for folks to give it a try and provide feedback. Enjoy!