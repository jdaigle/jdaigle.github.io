---
title: Introducing LightRail
layout: post
---

[LightRail](https://github.com/jdaigle/LightRail) is actually three different things:

1. An AMQP-based Message Broker for Windows (based on .NET)
2. An AMQP Client Library for .NET
3. A Service Bus Application Framework

The name "LightRail" is based on the transportation metaphor adopted by [MassTransit](https://github.com/MassTransit/MassTransit), another open source Service Bus framework. I also wanted the name to imply that it's a much smaller library and framework.

## A Quick History

Many years ago while working at [Clearwave](http://www.clearwaveinc.com) we decided to use [NServiceBus](https://github.com/Particular/NServiceBus) to build out our asynchronous processing components. NServiceBus is a great framework. At the time it really only targeted MSMQ, however we decided to build our service bus on top of [SQL Server Service Broker](https://msdn.microsoft.com/en-us/library/bb522893.aspx) to take advantage of the scaling and high availability that you get with SQL Server. It was relatively easy to implement *ITransport* back then, [and the code](https://github.com/jdaigle/servicebroker.net) is still kind of around.

Since then several things happened:

* NServiceBus went commercial. While still open source, it's not free to use in commercial applications. The [2.0 version](https://github.com/Particular/NServiceBus/tree/2.0) remained [open source with Apache V2](https://groups.yahoo.com/neo/groups/nservicebus/conversations/topics/9629).
* We quickly realized that NServiceBus' abstractions were too limiting for what we wanted to do with Service Broker.
* The 2.0 branch had a few performance issues.

So we actually "rewrote" NServiceBus as our own [CWService](https://github.com/clearwavebuild/CWServiceBus). It's kind of the same architecture, but very tightly coupled to Service Broker instead of MSMQ. And we made it open source.

Since I've been a consultant with [Intellinet](http://www.intellinet.com) have had the opportunity to work on a project that had a need for something extremely similar to what we built with CWServiceBus. **So I decided to start a project to rebuild CWServiceBus as a proper open source service bus application framework.**

But as I got started, I quickly realized that I was just rewriting NServiceBus... and I didn't want to do that.

## The AMQP Paradigm Shift

A few weeks ago I was listening to [this podcast](https://www.dotnetrocks.com/?show=1242) where they interviewed Clemens Vasters. They talked at length on his work with messaging standards and AMQP. Now I've known about AMQP from a very high level after having used [RabbitMQ](https://www.rabbitmq.com). But I didn't *really know AMQP* until I decided to dig into the [actual 1.0 specification](http://docs.oasis-open.org/amqp/core/v1.0/amqp-core-complete-v1.0.pdf). And I starting [learning even more]({% post_url 2016-01-21-amqp %}).

A few things struck me:

* RabbitMQ doesn't really support AMQP 1.0. It supports an older and incompatible version of the protocol.
* Azure Service Bus (mostly) supports AMQP 1.0. The "cloud" product is awesome, but the "on premise" version is... bad.
* There's *not really* any AMQP 1.0 based messaging broker for Windows today.

I was also been reading about [Disque](https://github.com/antirez/disque/blob/master/README.md) from Salvatore Sanfilippo (aka [Antirez](http://antirez.com/news/100)). It's a super-light-weight message broker based on his work with Redis. And then I got the bug. **I decided that I wanted to build a message broker for Windows using .NET and AMQP 1.0.** Crazy, I know.

As I mentioned at the top, LightRail is actually three different things:

**An AMQP-based Message Broker for Windows (based on .NET)**

I'm not doing a technical deep-dive with this article. But I'm planning to build something in .NET that runs on Windows. It'll be fast. The persistence model is sort of based on what Antirez has done with Disque/Redis. But the networking protocol will be AMQP. There may be a secondary management channel over HTTP for monitoring and managing the service.

I don't yet know what features it'll support. As a message broker it'll be similar to RabbitMQ, Azure Service Bus, and Disque. I.E. the client receives a message and must acknowledge after processing. There will be at-least-once semantics and possibly at-most-once. Topic/subscription queues. Dead-letter queues. TTL, retries, filtering, etc. AMQP supports transactions and so will LightRail.

The first version probably won't be distributed. But I may experiment with different topologies such as a master/slave replication or even multi-master for some scenarios.  

And maybe one day it'll be cross-platform targeting .NET Core. I could take advantage of [libuv](http://libuv.org/), similar to how Microsoft is building [Kestrel](https://github.com/aspnet/KestrelHttpServer), their cross-platform HTTP server.

**An AMQP Client Library for .NET**

The [Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) library has an AMQP support built in. But it's built for Azure Service Bus first-and-foremost. It's also not open source.

Microsoft also maintains a [AMQP 1.0 .NET Client Library](https://github.com/Azure/amqpnetlite) (aka "amqpnetlite"). Comparing the two, it's clear that they're *very similar*. Interestingly this library supports building both a "client" (which actively connects to a remote host) and a "server" (which listens for connections) or both.

While it is a fairly complete library and seems to be well testing and relatively mature, I'm not a huge fan. The API is rough. The various components (network, protocol, encoding) are all very tightly coupled. And it's not optimized for either client or server applications. And there may be performance related issues especially for server applications regarding allocations and value type boxing, among other concerns.

So I decided to build my own. I am borrowing quite a bit from amqpnetlite, especially around type encoding. But the "core" library is really just centered around encoding and framing. There will be separate client and server implementations.

**A Service Bus Application Framework**

An AMQP library is not enough for a Service Bus. For that you need to handle routing, message serialization, dispatch, topics/subscriptions, timeouts, etc. That's where the application framework comes in.

Now, instead of rewriting NServiceBus, I decided to take a different approach. Rather than being a service bus framework for building applications, *I'm building a framework for building a service bus*.

But what's the difference?

I believe that one of the drawbacks of NServiceBus (and MassTransit, et al.) is the idea of completely abstracting away the underlying message queue or broker. In the days of MSMQ, maybe that made more sense. MSMQ didn't really do anything other than queue and transport messages - the framework had to do a lot.

But the result of all this abstraction is that some platform specific features may get lost, unless they decide to add an abstraction for that feature. I think this is same sort of problem we find today with massive ORM frameworks and libraries. Everything is a [leaky abstraction](https://en.wikipedia.org/wiki/Leaky_abstraction). And this is the problem we ran into with SQL Server Service Broker.

So instead I want the framework to exist as a set of building blocks for building a service bus using some platform. Whether it's Azure, RabbitMQ, MSMQ, Server Broker, LightRail, whatever. So when you use it in your application, you're not using some generic service bus. You're using an 'X' service bus including whatever platform specific features it has. Any abstractions will be based conceptually on AMQP. Given that AMQP is a capital "S" Standard, I think it's a good move to try and view the world through that lens.

### Going Forward

Right now the project is very pre-alpha (or RC1 if I were the ASP.NET team ;). It barely does *anything* I described above.

In fact right now it's nothing more than a research project. Maybe one day it'll be mature enough to use in a real project. And then become a real open source project. Until then I'm kind of playing the the role of an [architecture astronaut](http://www.joelonsoftware.com/articles/fog0000000018.html). In the future I'll likely publish articles centered around specific technical components of the project.