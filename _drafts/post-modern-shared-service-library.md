---
title: Postmodern Shared Services
layout: post
---

In the classic "layered architecture" we have a prototypical service abstraction pattern which involves an interface and a concrete class. It's also known as the [facade pattern](https://en.wikipedia.org/wiki/Facade_pattern "facades"). For example:  

    public interface IShippingService
    {
        void Ship(int orderID);
        ShippingStatus GetShippingStatus(int orderID);
        List<int> GetShippedOrderIDS(DateTime from, DateTime to);
    }
    
    public class ShippingService : IShippingService
    {
        public void Ship(int orderID)
        {
            ...
        }
        
        public ShippingStatus GetShippingStatus(int orderID)
        {
            ...
        }
        
        public List<int> GetShippedOrderIDS(DateTime from, DateTime to)
        {
            ...
        }
    }
    
It doesn't get much simpler than this. It's relatively simple to consume, it's easy to mock for unit testing, and junior developers understand it.

But these facades *never stay simple*. As one starts building out the logic behind each method. 

1. **Dependency Injection**. In the Java world you would probably find a `ShippingServiceFactory` responsible for building the object. In the .NET world, we've fallen in love with [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection "dependency injection") frameworks and IoC containers to much chagrin.
2. **Aspect Oriented Programming**. 

### Dependency Injection

These facades are typically just a simple interface over a more complex set of objects including data access, IO and domain objects. These we typically need dependencies.

    public class ShippingService : IShippingService
    {
        public ShippingService(IOrdersRepository o, IEDI e)
        {
            ...
        }
    }

In this contrived example the `ShippingService` takes on two dependencies via the object's constructor. 