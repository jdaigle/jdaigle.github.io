---
title: ASP.NET Identity and OWIN Auth Code Review
layout: post
---

It's important to understand what ASP.NET Identity and what it is not. There are actually two separate projects: the [ASP.NET Identity project](https://aspnetidentity.codeplex.com/) and the [Katana project](https://katanaproject.codeplex.com/)(aka OWIN).

**Let's first break-down ASP.NET Identity**

This project is composed of three different assemblies:

1. Microsoft.AspNet.Identity.Core
1. Microsoft.AspNet.Identity.EntityFramework
1. Microsoft.AspNet.Identity.Owin

"Microsoft.AspNet.Identity.Core" assembly contains the "core" interfaces and base class implementations needed for Identity. E.g. `IUser`, etc. It also has implementations of `UserManager<TUser, TKey>` and similar. These "Manager" classes simply encapsulate common service behavior and depend on an `IUserStore<TUser, TKey>` implementation. The "core" assembly *does not* implement any of the `*Store` interfaces.

It also an internal `Crypto` implementation that uses the Rfc2898 hashing algorithm.

"Microsoft.AspNet.Identity.EntityFramework" is, as you might guess, just an implementation of `*Store` interfaces that uses EF6. When creating any of the concrete `*Store` classes, you supply a `DbContext` - the assembly ships with `IdentityDbContext` which maps the entities to the standard Identity databases schema. It's possible to subclass both the entity model objects and `IdentityDbContext` to provide your own additional implementation detail.

Finally, "Microsoft.AspNet.Identity.Owin". This is the bridge between OWIN (which I'll discuss in a bit) and ASP.NET Identity. The key bridge is `SignInManager<TUser, TKey>`: this class encapsulates the logic of validating user credentials against the `UserManager` *and* signing in the user view the OWIN `IAuthenticationManager`.

