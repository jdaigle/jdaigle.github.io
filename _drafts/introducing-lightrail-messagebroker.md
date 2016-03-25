---
title: Introducing LightRail Message Broker
layout: post
---

Before jumping in, I should point out at this point that **LightRail Message Broker is just an experimental research project**. It's not even close to production ready. And unfortunately I don't yet have a client project where I have a need to for it, so it may remain just a research project until then.

### What is LightRail Message Broker?

LightRail Message Broker is an open source message broker written in C# with a .NET AMQP client. This is a service which enables communication between processing using queues.

This is an asynchronous communication protocol. A process may send a message to a named queue, and only the processes waiting for messages from this queue will receive them.

**LightRail Message Broker** is a standalone service. But it also includes an **AMQP client library** (for .NET).

Additionally, I'm working on a separate application framework I'm calling **LightRail Service Bus**. This provides abstractions needed to build message-oriented applications and services adopting various patterns such as events, commands, publish/subscribe, etc. It also handles serialization and message dispatch. It's a lot like [NServiceBus](https://github.com/Particular/NServiceBus) or [MassTransit](https://github.com/MassTransit/MassTransit), except lighter-weight (in my opinion) and built first from AMQP.

The code is hosted on [GitHub](https://github.com/jdaigle/LightRail). Directions for building are in the repository.

### Why another Message Broker?

There's already a host of message broker and message queue systems available. Some purpose built like [Apache ActiveMQ](https://en.wikipedia.org/wiki/Apache_ActiveMQ), [IBM MQ](https://en.wikipedia.org/wiki/IBM_WebSphere_MQ), [MSMQ](https://en.wikipedia.org/wiki/Microsoft_Message_Queuing), and [RabbitMQ](https://en.wikipedia.org/wiki/RabbitMQ). More recently PaaS vendors started offering fully hosted systems; for example [Azure Service Bus](https://en.wikipedia.org/wiki/Microsoft_Azure) and [Amazon SQS](https://en.wikipedia.org/wiki/Amazon_Simple_Queue_Service). And finally there are a few hybrid systems such as those built into an RDBMS; a good example is [SQL Server Service Broker](https://en.wikipedia.org/wiki/Microsoft_SQL_Server#Service_Broker).

I had used many of these systems in the past. But I had never really done much in depth research into how they work and how they're built. But my recent deep dives into [AMQP]({% post_url 2016-01-21-amqp %}) really sparked a interest.

That's when I realized there is not a good example of a modern message broker or message queue written in C# or .NET. Most of the open source versions are built on Java.

### Let's talk Architecture

The server speaks AMQP. The AMQP stack basically involves maintaining a *connection* between hosts. Individual AMQP frames are sent from one connection and received by another. An AMQP *connection* can multiplex multiple *sessions*. An AMQP *session* can be thought of as a bidirectional channel between two applications. It's neat that AMQP supports multiplexing; this enables a application to have multiple messaging streams over a single TCP connection, which may improve network overhead and performance. An AMQP *session* then can multiplex multiple unidirectional *links*. An AMQP *link* is the logical association to a specific target, such as a queue. *Links* are unidirectional; they're attached for either sending or receiving.

These logical AMQP objects create the state machine which drives the messaging protocol.

In this implementation, the server starts up an TCP listener thread. I am taking full advantage of the [I/O Completion Port API](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365198.aspx) built into Windows.

- Each time a socket is accepted, we create an AmqpConnection instance which starts the state machine.
- The AmqpConnection constantly receives data from the underlying socket in a loop. We're looking for AMQP frames, which always begin with a 4 byte preamble indicating the total length of the frame. Each frame is processed via the state machine; handling beginning/ending session, attaching/detaching links, etc. And also receiving transfer frames (messages).
- All writes to the underlying socket are buffered in the application. Writes are enqueued in a ConcurrentQueue and flushed. A RegisteredWaitHandle is signaled on each Enqueue(). An atomic register ensures only a single thread is actively looping over the queue and writing the queued writes to the underlying socket. The worker thread returns when no more queued writes exist.
- The buffering of writes allows the system to continue receiving data while asynchronously writing. The thread handling a received frame must not block! That means I/O operations should be queued. And all shared state should be non-blocking, non-locking. Ideally wait-free, but that's harder to do.