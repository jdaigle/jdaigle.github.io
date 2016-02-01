---
title: High Performance Message Broker Design
layout: post
---

In my [previous post]({% post_url 2016-01-25-introducing-lightrail %}) I talked of beginning research into designing and implementing an AMQP-based message broker.

Implementing the network layer and AMQP protocol is relatively easy. But the actual message broker implementation (queuing, dispatch, and persistence) is an interesting problem.

Here are some of the requirements:

* I should be able to concurrently enqueue messages.
* Enqueued messages may have a TTL.
* Instead of de-queuing from the *head* of the queue, I need to be able to *ACQUIRE* a message.
* An *ACQUIRED* message will be delivered. If accepted, the message will be *ARCHIVED*. If released the message becomes *AVAILABLE* for delivery again from the same position (i.e. the head).
* I should be able to spontaneously release *ACQUIRED* messages after an expiry.
* Consumers should be notified of new messages for delivery.
* Consumers may filter messages.
* Consumers should implement flow control to restrict the number of delivered messages.

I took a lot of inspiration from other open source projects such as [Apache QPID](http://qpid.apache.org/), [Apache ActiveMQ](http://activemq.apache.org/) (specifically [ActiveMQ Apollo](https://activemq.apache.org/apollo/)), [RabbitMQ](https://en.wikipedia.org/wiki/RabbitMQ), and [Azure Service Bus](https://azure.microsoft.com/en-us/services/service-bus/). And the [AMQP Spec](http://docs.oasis-open.org/amqp/core/v1.0/amqp-core-complete-v1.0.pdf) itself lends some guidance from the protocol level of how a broker might behave.

### Concurrent Linked List Queue

What I've discovered is that a traditional queue, such as .NET's *ConcurrentQueue<T>* is insufficient. Particularly due to acquiring and releasing messages. But I can use a concurrent linked list, with some modifications.

    [HEAD] -> [ ] -> [ ] -> [ ] -> [ ] -> [ ] -> ... -> [ ] -> [TAIL] -> NULL

Using a linked list as a queue requires maintaining `HEAD` and `TAIL` pointers. Entries are enqueued by a) creating a new entry b) setting the `next->` pointer of the current `TAIL` to the new node and c) and setting `TAIL` to point to the new node. To dequeue a) take the `HEAD` and if `next->` is not NULL b) set `HEAD` to point to the next node.

To achieve concurrency *without blocking or locking*, we can use [compare-and-swap instructions](https://en.wikipedia.org/wiki/Compare-and-swap). Essentially we loop until the system can atomically swap pointers from a known state. Here's the implementation for Enqueue():

<script src="https://gist.github.com/jdaigle/cc1449a99d4e6448672d.js?file=enqueue.cs"></script>

We don't actually want to "dequeue" in the sense of *removing* the node. Instead we want to *acquire*. We can accomplish this by maintaining some additional state in the queue entry itself. Consider a `state` field with three values: `AVAILALBE`, `ACQUIRED`, and `ARCHIVED`.

At any point we can get the next *available* node by starting at the `HEAD` and walking the list until we reach an `AVAILABLE` entry, or NULL. Then *acquire* the entry by changing the state. The entry can be `released`, causing re-delivery, or `ARCHIVED`. Again, we can use an algorithm that makes use of compare-and-swap to do this concurrently without blocking or locking.

When an entry is `ARCHIVED`, it's not immediately removed the linked list. Instead we occasionally run a tweak-able scavenging algorithm. This algorithm walks the linked list and removes `ARCHIVED` entries.

### Queue Consumers

In a message broker consumers of a queue usually come and go over. There may be multiple consumers. And we need a way to efficiently deliver messages to attached consumers.

In this case a consumer will ultimately be some remote link (that's AMQP terminology) that wishes to receive messages from a queue. So when a remote client connects, we may create and associate consumer for a queue. The consumer is disposed with the connection/session/link (actually AMQP supports re-attaching broken links by exchanging unsettled state maps, but it's not a requirement).

If I were building a simple blocking queue for a single consumer, I would utilize `System.Threading.AutoResetEvent`. The consumer has an event loop which attempts to dequeue the next message, and if unsuccessful calls `WaitOne()` on the WaitHandle. Each time a message is enqueued, I call `Set()` on the WaitHandle which signals the *blocked* thread to continue.

In theory I could create a blocking thread for each consumer. But there are several problems with this approach. 1) Each blocked thread consumes memory, and occupies a ThreadPool thread that could otherwise be doing work. 2) The WaitHandle will only a signal a single thread. So only a single consumer will be signaled, and only that consumer will attempt delivery.

I need to be able to signal *all consumers*. And do so fairly.

My research lead me to implement a 
*single message delivery event loop thread*. When signaled, it will loop over each consumer and attempt delivery of a single message. And continue until all messages are delivered.

While this does in fact work, there are still concerns. a) What if there are messages to deliver but no consumers or b) what if there are consumers but no messages to deliver. One could visualize a situation where the loop is running constantly but not actually doing anything. Thus starving the system of resources.

Instead, we can have the loop *end* at the point of either a) no subscribers or b) no messages. We need a way to restart the loop. That's when I learned about `ThreadPool.RegisterWaitForSingleObject`. This allows us to register a WaitHandle which calls a callback delegate when the WaitHandle is set. Instead of having one of our own threads blocked, the OS efficiently handles it for us.

Each time `Set()` is called, it will call our callback delegate. I needed to implement a small flag to track whether the loop is already running to prevent it from running concurrently. We return from the callback when we're out of work to do.

Here's the implementation:

<script src="https://gist.github.com/jdaigle/cc1449a99d4e6448672d.js?file=queuePump.cs"></script>

### Flow Control

Another aspect of the message broker is the idea of flow control. Each consumer implements credit-based flow control. Or, in AMQP terms, a link-credit. The "credit" is decremented on each delivery until it reaches 0. At 0, no more messages may be delivered until the credit increases.

This ensures we never deliver more messages than the link can handle.

This is actually quite straightforward to implement in our broker. The consumer object maintains the credit, and is decremented on each delivery. Before each delivery, the credit is inspected to ensure we can deliver. If not, that consumer is skipped. The consumer's credit can be reset after messages are processed, etc. In the case of AMQP, we'll receive `FLOW` frames with updated credit.

However the above message event loop will need to be modified to account for credit. Specifically, when no consumers have available credit we can stop the event loop. Upon setting new available credit, we can again `Set()` the WaitHandle to restart the event loop.

**Next time** I'll talk about making queues durable with append-only-file (AOF) style transaction logs, and idea I borrowed from Redis;.