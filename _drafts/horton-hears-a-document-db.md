---
title: Horton Hears a Document DB
layout: post
---

[Document-oriented database](https://en.wikipedia.org/wiki/Document-oriented_database) models are gaining increased popularity with developers. In particular we see dedicated document databases such as MongoDB and RavenDB, and online PaaS offerings from Amazon, Microsoft, and Google.

At its core a document-oriented database is simply a key/value store. The value, in this case, is a usually a serialized "document" with a loose schema. For example, XML or JSON. And most implementations provide ACID transactions around *just a single document*.

Many implementations also include the ability index, map, and/or query based on the *content* of the document. These implementations are aware of the particular document serialization and contain native methods to parse and query. Some expose custom query DSL. Others incorporate and expose search engines such as [Lucene](https://lucene.apache.org/). Some implementations also incorporate server-side [MapReduce](https://en.wikipedia.org/wiki/MapReduce) engines.

But many of these features are either included with, or can be built upon, the powerful relational databases that many of us use everyday. I was inspired by Jeremy Miller's recent exploration into using [Postgresql as a Document Db for .Net Development](http://jeremydmiller.com/2015/10/21/postgresql-as-a-document-db-for-net-development/). For a previous project, I built a document database on top of Microsoft SQL Server using its XML data type. I built a small client library that worked like an ORM exposing a unit of work and encapsulating the work of querying, change-tracking, and serialization.

I decided to resurrect the idea as a proof-of-concept.

## HortonDB: Proof of Concept

[HortonDB](https://github.com/jdaigle/hortondb) is a proof-of-concept .NET library that uses Microsoft SQL Server and XML data as a document database. The code is open source and hosted on GitHub.

**Why XML over JSON?** The simple answer is that SQL Server currently has decent support for XML and not JSON. But XML isn't so bad. It's widely supported by virtually all platforms and frameworks, and has pretty good tooling. SQL Server, in particular, has support for:

* Schema Validation
* 

### Features

Since this is just a proof-of-concept, I didn't include every feature possible.

* **Unit of Work.** There's an `IDocumentSession` which is responsible for tracking changes all loaded documents, new documents, and deleted documents. Changes are persisted back to the database when the user calls `SaveChanges()`.
* **Identity Map.** The identity map provides the first-level cache for instance of a document session. This allows one to `Load()` a document multiple times during a unit of work and always return the same instance. It is also responsible for maintaining the dictionary of the internal metadata required for the library.

**Not Implemented Features:**

* **Load by IDs**. Quickly load multiple documents (perhaps in batches?) given a set of IDs.
* **Querying.** The library only supports simple CRUD operations. Building a querying API is tricky business which I'll talk about in a bit.
* **Result Caching.** Also known as a second-level cache or aggressive cache. Retrieving a single document and de-serializing it into an object is an expensive operation. For documents that are frequently accessed, but rarely change, we can cache the object instance and return that instance on `Load()` until we detect the data has changed in the database. Because it's a single copy of an object, generally this object should be used "read-only". This could be implemented with SQL Server's rowversion column.
* **Read-only Documents**. Simply, documents that should not change and thus do not need to be tracked.
* **Bulk Insert**. 

### Queries

SQL Server has native support for query XML data, along with other operations. I won't go into a lot of details as it's [fairly well document](https://msdn.microsoft.com/en-us/library/ms190798.aspx). Natively, SQL Server uses a query language called [XQuery](https://msdn.microsoft.com/en-us/library/ms189075.aspx).

### Indexing

partial decomposition into relational? (persisted computed columns)

[Promote Frequently Used XML Values with Computed Columns](https://msdn.microsoft.com/en-us/library/bb500236.aspx)

### JSON Support

## Further Reading / References

* [Sql Server XML columns substitute for Document DB?](http://stackoverflow.com/questions/5119660/sql-server-xml-columns-substitute-for-document-db).
* Jeremy Miller has recently starting building an OSS project around using [PostgreSQL as a Document Db for .Net Development](http://jeremydmiller.com/2015/10/21/postgresql-as-a-document-db-for-net-development/).
* Rob Conery has [done the same thing](http://rob.conery.io/2015/08/20/designing-a-postgresql-document-api/).