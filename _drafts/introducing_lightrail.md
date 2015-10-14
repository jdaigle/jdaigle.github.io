---
title: Introducing LightRail
layout: post
---

[LightRail](https://github.com/jdaigle/LightRail "LightRail") is an opinionated message broker and enterprise service bus (ESB) framework designed for .NET server applications. It is based on the same architecture principals popularized by the successful [NServiceBus](https://github.com/Particular/NServiceBus) and
[MassTransit](https://github.com/MassTransit/MassTransit) projects.

LightRail originated as a streamlined service bus framework based on the architecture of NServiceBus 2.x. I decided to rewrite it as an open source project that I can incorporate in future projects. Additionally I wanted to experiment with a few new architectural patterns. 

### Why would I build a new framework?

I'm a *huge* fan of NServiceBus; it's foundations created by the smart and talented [Udi Dahan](https://twitter.com/UdiDahan). I was an early adopter of NServiceBus while it was still open source and *free-as-in-beer*. I even wrote an ITransport implementation for NServiceBus 2.x for SQL Server Service Broker.

While work at [Clearwave](http://www.clearwaveinc.com), I experimented heavily building very light-weight service bus implementations usually involving Service Broker. These were all heavily influenced by NServiceBus both architecturally and from a developer use-case perspective.

Overtime I realized that I could build an NServicBus clone and enhance it with the features *I want* and remove the features I don't need. At the end of the day, I ended up building a streamlined framework that's been used to power the back end of Clearwave's software for the past 6 years.

### Side Note: Why do I like SQL Server Service Broker?

1. If you're already using MS SQL Server, you get it for free.
2. Once your SQL Server instance is highly available, so are your queues.
3. It scales rather well.
4. It solves the load balancing problem that occurs with MSMQ. If you read about NServiceBus' solution, it involves complex choreography around a "Distributor". But it has severe limitations in handling offline servers - messages can be significantly delayed or even lost!

MSMQ is not a centralized queue store. In order to distribute your message handling, MSMQ must store and forward messages to other computers. 

MSMQ certainly serves a purpose. And it works very well for that use case. However I've found that having a centralized queue store with remote readers is a far simpler topology for many applications.

But you have to be careful with Service Broker. There is very little useful documentation, and very few people blog about it. I recommend reading [Remus Rusanu's](http://rusanu.com/) blog and articles. On example is that I go against his, and essentially Microsoft's guidance, regarding using Service Broker in a fire-and-forget pattern - there are a lot of gotcha's that one must watch for both during development and in operation. But it absolutely can be made to work.

**Note to self:** I really should learn more about [RabbitMQ](https://www.rabbitmq.com/) and [AMQP](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol).

### What's special about LightRail?

Besides being a really small framework, I've made a few interesting architectural decisions.

#### JSON as the default message serialization format.

XML is out, JSON is in. It's possible to write really fast XML serializers. But XML is overkill for the types of messages usually implemented by developers. Usually our messages are simply a .NET class with a set of properties. Modern JSON serializers are very fast.

#### Classless message handlers.

In almost any service bus framework you'll find this interface:

    public interface IMessageHandler<TMessage>
    {
        void Handle(TMessage message);
    }

The developer will write a class that implements this interface. The framework scans assemblies and finds types that implement this interface and maps it to the message type. When processing a message, the framework usually uses an IoC or Service Locator to get a new instance of the class, usually with dependency injection, and then dispatch a call the method passing the message.

This is a simple pattern to grok, but it's full of boilerplate! I have to write a constructor that accepts dependencies and writes to instance fields. The framework needs to allocate a unique instance of the class with the dependencies fulfilled. And then it just executes a single ***function***. For example:

    public class MyMessageHander : IMessageHandler<HelloWorld>
    {       
        public MyMessageHandler(IDependency d1, IOtherDependency d2)
        {
            _d1 = d1;
            _d2 = d2;
        }
    
        private readonly IDependency _d1;
        private readonly IOtherDependency _d2;
    
        public void Handle(HelloWorld message)
        {
            ...   
        }   
    }

The keyword in the previous paragraph is ***function***. This class doesn't, and really shouldn't maintain any additional state. Because from the service bus framework's perspective, it's not likely to live longer than the current message's context.

And no, setter injection is not the right solution!

What if our message handler was just a static function? And, in addition to the TMessage parameter we simply include any required dependencies? What if this was our message handler?

    public static class DoesntMatter
    {
        [MessageHandler]
        public static void Handle(HelloWorld message, IDependency d1, IOtherDependency d2)
        {
            ...   
        }   
    }

That does the _exact same thing_ but is so much cleaner in my opinion.

That's what LightRail does. It implements some clever tricks to detect and inject the dependencies when calling the function. Also, it currently requires the usage of an attribute to assist the framework when scanning assemblies for message handler functions. I don't love it, but I have found a better way yet to mark a function as a message handler (versus a helper method or something else). 