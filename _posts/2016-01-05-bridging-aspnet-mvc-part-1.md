---
title: Bridging ASP.NET MVC Part 1 - Introduction  
layout: post
date: 2016-01-05 08:00
---

This is **part 1** of a multi-part series on "fixing" some of the inherit design problems with ASP.NET MVC.

- [Part 1 - Introduction]({% post_url 2016-01-05-bridging-aspnet-mvc-part-1 %})
- [Part 2 - The View Engine]({% post_url 2016-01-05-bridging-aspnet-mvc-part-2-the-view-engine %})
- Part 3 - View Model Conventions
- Part 4 - Routing and URL generation
- Part 5 - The Controller Factory
- Part 6 - ActionResults and Content Negotiation

**ASP.NET MVC** has been the contemporary framework of choice for backend web development on the .NET stack almost since its first release around 2008-2009. And now here we are, seven years later, and by-and-large the framework's programming patterns have remained relatively unchanged.

At the time of it's initial development the hottest and trendiest web development stack was [Ruby on Rails](https://en.wikipedia.org/wiki/Ruby_on_Rails). So it's no surprise that Microsoft borrowed a great many of Rails' conventions when designing ASP.NET MVC. Whether good or bad.

Over the years people have discussed and debated the patterns (and anti-patterns?) and other short comings of the framework. In fact, "alternative" frameworks such as [FubuMVC](http://fubuworld.com/fubumvc/) sprung up as a result. Unfortunately, virtually all of these "alternative" frameworks see little to no adoption.

**Arguably the worst part of being a .NET developer is the dependence on and  suffocation by "1st party" frameworks and tools.** For many dev shops, "3rd party" and open source is never considered an option or is simply forbidden; it's either built by Microsoft or it doesn't exist! I think the .NET development world is, in a bad way, unique in this aspect.

Side note: more recently there's been a rise in [Sinatra](https://en.wikipedia.org/wiki/Sinatra_(software)) style frameworks such as [Nancy](http://nancyfx.org/). I *really like* NancyFX. And I hope they stay strong.

So while I do have many of my own ideas for how I would refactor ASP.NET MVC into something better (in my opinion), I know it'll see *zero adoption*. And it will also be met with heavy resistance for new projects I work on.

Instead, I'm going to show a few tricks I've learned over the past seven years that make ASP.NET MVC much smarter. Over the course of this series I'll discuss an aspect of the framework I dislike, and show how to make it better. I may also add to the series as I learn new tricks, and as the framework itself evolves.