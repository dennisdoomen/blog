---

title: "Principles for Successful Package Management"
excerpt: "How applying the SOLID principles on package design can help prevent the dependency hell"
date: '2016-10-30T19:24:00.001+01:00'

tags:
- components
- Architecture
- nuget
- package-management
---

A couple of months ago I [shared](https://www.continuousimprover.com/2016/05/the-magic-of-hiding-your-nuget.html#uds-search-results) some tips & tricks to help you prevent ending up in NuGet dependency hell. As a big fan of the [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles, I've always wondered why nobody thought of applying these principles on the package level. If SOLID can help you to build cohesive, loosely coupled components which do one thing only and do that well, why can't we do the same thing on the package level. As it happens, my colleague [Jonne](https://twitter.com/jonnekats) enthousiastically referred me to the book [Principles of Package Design](https://leanpub.com/principles-of-package-design) by [Matthias Noback](https://twitter.com/matthiasnoback?lang=nl). It's available from Leanpub and does exactly that, offering a couple of well-named guidelines inspired by SOLID that will help you design better NuGet, NPM or whatever your package management solution of choice uses.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2016-10-30/principles-of-package-design.jpg" class="align-center" />

The first half of the 268 pages provide an excellent refresh of the SOLID principles. He even does a decent job of explaining the inversion of control principle (although I would still refer to the [original](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=asap_bc?ie=UTF8) to really grasp that often misunderstood principle). After that he carefully dives into the subtleties of cohesion as a guiding principles before he commences on the actual package design principles. The examples are all in PHP (yeah, really), but the author clearly explains how these would apply to other platforms. Notice that this post is mostly an exercise for me to see if I got the principles right, so I would highly recommend buying the .epub, .mobi or PDF from Leanpub. It's only 25 USD and well worth your money. So let's briefly discuss the actual principles.

**The Release/Reuse Equivalency Principle**

IMHO, the first principle has a rather peculiar name. Considering its purpose, it could have been called The Ship a Great Package Principle. The gist of this principle is that you should not ship a package if you don't have the infrastructure in place to properly support that. This means that the package should follow some kind of clear (semantic) versioning strategy, has proper documentation, a well-defined license, proper release notes, and is covered by unit tests. The book goes into great lengths to help you with techniques and guidance on ensuring backwards compatibility. Considering the recentness of the book and the fact it mentions Semantic Versioning, I would expected some coverage of [GitFlow](https://www.continuousimprover.com/2015/04/software-versioning-without-thinking.html) and [GitHubFlow](https://www.continuousimprover.com/2015/04/software-versioning-without-thinking.html). Nonetheless, most of the stuff mentioned here should be obvious, but you'll be surprised how often I run into a unmaintainable and undocumented package.

**The Common Reuse Principle**

The purpose of the second principle is much clearer. It states that classes and interfaces that are almost always used together should be packaged together. Consequently, classes and interfaces that don't meet that criteria don't have a place in that package. This has a couple of implications. Users of your package shouldn't need to take the entire package if they just need a couple of classes. Even worse, if they use a subset of the package's contents, there must not be a need to get confronted with additional package dependencies that have nothing to do with the original package. And if that specific package has a dependency, then it's an explicit dependency. A nice side-effect of this principle is that it makes packages Open for Extension and Closed for Modification.

I've seen packages that don't seem to have any dependencies until you use certain classes that employ dynamic loading. [NHibernate](http://nhibernate.info/) is clear violator of this principle in contrast to the well-defined purpose of the [Owin](https://www.nuget.org/packages/Owin/) NuGet package. My own open-source library, [Fluent Assertions](https://www.fluentassertions.com/) also seems to comply. When a contributor proposed to build a Json extension to my library, I offered to take in the code and ship the two NuGet packages from the same repository. So if somebody doesn't care about Json, it can use the core package only, without any unexpected dependencies on NewtonSoft.Json.

**The Common Closure Principle**

The third principle is another one that needs examples to really grasp its meaning. Even the definition doesn't help that much:

> _The classes in a package should be closed against the same kinds of changes. A change that affects a package affects all the classes in that package._

According to many examples in the book, the idea is that packages should not require changes (and thus a new release) for unrelated changes. Any change should affect the smallest number of packages possible, preferably only one. Alternatively, a change to a particular package is very likely to affect all classes in that package. If it only affects a small portion of the package, or it affects more than one package, chances are you have your boundaries wrong. Applying this principle might help you decide on which class belongs in which package. Reflecting on Fluent Assertions again, made me realize that even though I managed to follow the Common Reuse Principle, I can't release the core and Json packages independently. A fix in the Json package means that I also need to release the core package.

**The Acyclic Dependencies Principle**

For once, the fourth principle discussed in this book is well described by its definition:

> _The dependency structure between packages must be a directed acyclic graph, that is, there must be no cycles in the dependency structure._

In other words, your package should not depend on a package which dependencies would eventually result in cyclic dependency. At first thought, this looks like an open door. Of course you don't want to have a dependency like that! However, that cyclic dependency might not be visible at all. Maybe your dependency depends on something else that ultimately depends on a package that is hidden in the obscurity of all the other indirect dependencies. In such case, the only way to detect that, is to carefully analyze each dependency and create a visual dependency graph.

Another type of dependencies that the book doesn’t really cover are diamond dependencies (named for the visual dependency graph). Within the .NET realm this is a quite a common thing. Just consider the [enormous amount](http://nugetmusthaves.com/Dependencies/Newtonsoft.Json) of NuGet packages that depend on NewtonSoft's Json .NET. So for any non-trivial package, it's quite likely that more than one dependency eventually depends on that infamous Json library. Now consider what happens if those dependencies depend on different versions.

The book offers a couple of in-depth approaches and solutions to get yourself out of this mess. Extracting an adapter or mediator interface to hide an external dependency behind is one. Using inversion-of-control so that your packages only depend on abstract constructs is another. Since the book is written by a PHP developer, it's no surprise that it doesn't talk about ILMerge or its open-source alternative [ILRepack](https://github.com/gluck/il-repack). Both are solutions that will merge an external .NET library into the main DLL of your own package. This essentially allows you to treat that dependency as internal code without any visible or invisible DLL dependencies. An alternative to merging your .NET libraries is to use a source-only NuGet package. This increasingly popular technique allows you to take a dependency on a NuGet package that only contains, surprise, source code that is compiled into your main package. [LibLog](https://github.com/damianh/LibLog), [TinyIoc](https://github.com/grumpydev/TinyIoC) and even my own caching library [FluidCaching](https://github.com/dennisdoomen/FluidCaching) uses this approach. It greatly reduces the dependency chain of your package.

**The Stable Dependencies Principle**

The name of the principle is quite self-explanatory, but the definition is even clearer.

> _The dependencies between packages in a design should be in the direction of the stability of the packages. A package should only depend upon packages that are more stable than it is._

In other words, you need to make sure you only depend on stable packages. The more stable your dependency, the more stable your package is going to look to your consumers. Determining whether a package is stable or not isn't exact science. You need to do a bit of digging for that. For instance, try to figure out how often a dependency introduced a breaking change? And if they did, did they use [Semantic Versioning](http://semver.org/) to make that clear? How many other public packages depend on that package? The more dependents, the higher the chance that the package owners will try to honor the existing API contracts. And how many dependencies does that package have? The more dependencies, the higher the chance some of those dependencies introduce breaking changes or instability. And finally, check out its code and judge how well that package follow the principles mentioned in this post? The book doesn't mention this, but my personal rule-of-thumb to decide on whether I will use a package as a dependency is to consider the circumstances when the main author abandons the project. The code should either be good enough for me to maintain it myself/ourselves, or the project should be backed by a large group of people that can ensure continuity.

**The Stable Abstractions Principle**

Now if you understand (and agree) with the Stable Dependencies principle, you'll most definitely understand and agree with the Stable Abstractions Principle. After all, what's more stable? An interface, an abstract type or a concrete implementation? An interface does not have any behavior that can change, so it is the most stable type you can depend on. That's why a well-designed library often uses interfaces to connect many components together and quite often provides you would with an interface-only package. For the same reason, the Inversion of Control principle tries to nudge you in the same direction. In fact, in the .NET world even interfaces are being frowned on and are being replaced with old-fashioned delegate types. These represent a very tiny and very focused interface so it doesn't get any more stable than that. And because of their compatibility with C#'s lambda statements you don't even need to using a mocking library.

**So what about you?**

The names are not always that catchy and easy to remember, mostly because they use the same wording, but the underlying philosophy makes a lot of sense to me. I've already started to re-evaluate the design decisions of my projects. The only thing I was hoping to read more about is the explicit consequence of building a component or package as a library versus building it as a framework. This is something that heavily influences the way I'm building [LiquidProjections](https://liquidprojections.net/), my next open-source project.

So what do you think? Do you see merits in these principles? Do they feel as helpful as the original SOLID principles? I've love to know what you think by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better designs.