---
layout: post
title: Website Status Checker for Integration Tests
---

During some recent client work I faced some issues after a deployment verifying that all the Http/Https bindings and url redirects were working as expected. As these were not part of any regular testing plan, I couldn't provide the level of assurance I'd have liked that they had worked before. This combined with previous trouble tracking expiry of client SSL certificates prompted me to think that it would be useful to be able to include these tasks in integration tests.

While there are probably services that offer these functions, I was wanting a solution that can added into an existing testing suite, and doesn't require signing the client up to any additional services or running any additional services on the client servers.

The tests I'd require this to perform are:
- Verify that the DNS can be resolved
- Ping the server to verify that it's online
- Make a web call to the home page of the website, and verify that it returns a 200 OK
- Retrieve the certificate for the site, and verify the expiry date

For the implementation I wanted to go for a simple fluent api to make it easy to use, and use [Fluent Assertions](http://fluentassertions.com/) internally as a way to make my test class compatible with most common testing frameworks. The rest of the code is just normal .net, so doesn't bring in any other dependencies aside from Fluent Assertions.

The following code snippet shows the currently implemented methods:
```csharp
SiteChecker.ForDomain("anarks2.com")
                .WithProtocol("http")
                .AssertThatResolvesDns()
                .AssertIsOnline()
                .AssertIsAccessible()
                .AssertRedirectsTo("blog.anarks2.com", "https")
                .AssertCertIsValidFor(TimeSpan.FromDays(30));
```

Nuget Package: https://www.nuget.org/packages/SiteStatusChecker/1.0.0
Source Code: https://github.com/Togusa09/SiteStatusChecker