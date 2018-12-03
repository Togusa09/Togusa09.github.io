---
layout: post
title: Unit testing around HttpContext
categories: c# HttpContext Webforms ASP.net 
---

While testing around web frameworks has greatly improved in recent years, but when there is a need to work with older frameworks like WebForms, the story isn't quite as friendly. As webforms was not designed with testability or dependency injection in mind, it has a number of components that are statically accessed and can't be overriden for testing. A central one of these is the `HttpContext`, which holds the current user identity and session information

## Retrieving Http Context ##

- Static resolution - The current HttpContext can be retrieved by using the `HttpContext.Current` static method. This is one of the commonly used older methods, but should be avoided as it makes testing _extremely_ impractical.
- Dependency injection - Some Dependency Injection frameworks provide integration with WebForms to provide dependency injection via Property Injection. ie [Property Injection](https://autofaccn.readthedocs.io/en/latest/integration/webforms.html#structuring-pages-and-user-controls-for-di)
- Static Dependency resolution - For situations where property or constructor based dependency injection are not available, a globally accessible container instance can be used to resolve. ie [Resolving global container](https://autofaccn.readthedocs.io/en/latest/integration/webforms.html#manual-property-injection)

## Overriding in tests ##

### Setting Current HttpContext ###

While the old `HttpContext` class can't be overrwitten, it does have some flexibility in testing, by setting properties and explicitly setting `HttpContext.Current`. This allows some things, such as the user principal to be manually set, but it is not possible to set properties like Session, so this a limited solution. This is also a bad idea for unit tests, as it is a singleton instance, and you don't want tests interfering with each other.

Unfortunately as `HttpContext.Current` is a `HttpContext`, it is not possible to set it to a mocked instance of `HttpContextBase`.

### Mocking HttpContextBase ###

Two additional classes have been made available since .net 3.5 to assist with working around the HTTPContext.

HttpContextBase - Abstract base for HttpContextWrapper. It is designed to be overridable to allow for mocking in tests. This class is also used for exposing the HttpContext in newer frameworks like MVC. The `Autofac.Web` library adds a registration for this class to the request lifetime scope for dependency injection.

HttpContextWrapper - Wrapper class that takes the old HttpContext as a parameter. It then exposes all the same properties as HttpContext, but inherits from HttpContextBase. It can then be used to transform from `HttpContext` to `HttpContextBase` via `new HttpContextWrapper(HttpContext.Current)`. As manually converting to HttpBase may still rely on `HttpContext.Current`