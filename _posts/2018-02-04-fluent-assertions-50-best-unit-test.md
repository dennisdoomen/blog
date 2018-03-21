---
title: 'Fluent Assertions 5.0: The best unit test assertion library in the .NET realm
  just got better'
date: '2018-02-04T14:37:00.001+01:00'
tags:
- fluentassertions
modified_time: '2018-02-04T14:37:32.169+01:00'
thumbnail: https://lh3.googleusercontent.com/-ObCg1CVLz2c/WncMmgez64I/AAAAAAAANwE/oU7Ym-JoF9cyy62GtYLNZo15M1zrYFAegCHMYCw/s72-c/clip_image001_thumb3?imgmax=800
blogger_id: tag:blogger.com,1999:blog-15137028.post-8455238901380086634
blogger_orig_url: http://www.continuousimprover.com/2018/02/fluent-assertions-50-best-unit-test.html
---

It has been almost a year since [version 4.19](https://github.com/fluentassertions/fluentassertions/releases/tag/4.19.0), the last functional release of Fluent Assertions was shipped. Not because of a lack of feature requests, but simply because this new version has cost me all the private time I had. My main goal of this release was to repair some of the design mistakes I have made over the years and introduce the only key feature that Fluent Assertions was still missing compared to other libraries. This also gave me the time to run a contest resulting in a great new logo designed by [Ben Palmer](https://github.com/IUsername). So after three betas and five release candidates, I present to you [Fluent Assertions 5.0](https://github.com/fluentassertions/fluentassertions/releases/tag/5.0.0). It contains loads of new features, small and big, but also tries to break with the past.

![logo image]({{ "/assets/images/posts/2018-02-04_FluentAssertions_NewLogo.png" | absolute_url }})

#### Embracing standards

Over the years, I've been using different techniques to support multiple platforms. I started with using Linked Files to share files between multiple versions of the main project. This worked, but it subdued any attempts to keep aggressively refactoring my code. Moving files around doesn't work well if you have five links to that same file. Then, with Visual Studio 2013 (I think), we got Shared Projects. This allowed me to refactor away and use conditional symbols to share the same files with the platform-specific projects. The next innovation that happened in the .NET space was the Portable Class Library. With the help of [Oren Novotny](https://oren.codes/) (who is a master in anything .NET), we refactored the code-base to employ a mechanism where the bulk of the code was in a single PCL assembly and the platform-specific stuff would go in a smaller platform-specific assembly. At run-time, it used a [bait-and-switch](https://oren.codes/tag/pcl/) mechanism to dynamically load the platform-specific assembly and connect the implementation classes to the interface hooks the core assembly offered. In a way, it was doing dependency-injection.

However, all of this is in the past, now that we have .NET Standard and cross-compilation. With this release, Fluent Assertions is build from a single project that targets .NET Standard 1.4, 1.6 and 2.0, as well as the full .NET 4.5 Framework. You might wonder why I target multiple versions of .NET Standard. The simple reason is that .NET Standard 1.4 doesn't support all the features of the .NET Framework. The higher the .NET Standard version, the more features will light up. A nice side-effect of all of this is that it's now also much easier to contribute to this little project of mine (yes, that's a hint).

#### Moving towards a unified API

One of the things that has annoyed me for years is the inconsistency of the API. This all started when I introduced this very powerful and useful API for comparing deep object graphs, `ShouldBeEquivalentTo`. I really liked to be able to use the type of the subject for nice fluent expressions. I needed access to the generic type parameter representing the subject-under-test. I could not simply define another `Should<T>()` method since the compiler prefers that overload over `Should<T> where T : IEnumerable<TItem>`. That's why I settled for `ShouldBeEquivalentTo`. This caused a lot of confusion, especially since there was already a `Should().BeEquivalentTo()` on collections. I've tried to change that in a non-breaking way a couple of times, but it always resulted in a suboptimal experience.

In 5.0, I made several behavioral changes (more on that later) that allowed me to finally align all assertions. You'll now find that all assertions start with `Should()`, e.g.

* `object.Should().BeEquivalentTo(anotherObject)`
* `action.Should().Throw<MyException>()`
* `func.Should().NotThrow()`
* `monitoredObject.Should().Raise("Event")`
* `executionTime.Should().Exceed()`

You may wonder about that existing `Should().BeEquivalentTo()` that was available to collections. Well, that's now behaving as a deep, structural comparison of collections. Existing calls will do exactly what it did before. It'll just give you more information when the items in one collection don't match those in the other collection.

#### Subject Identification

As a passionate open-source developer you keep an eye on your competition, in particular if that competition has features that make developers choose for the competition. Now, Fluent Assertions has grown up to be a mature and complete library and there's not a lot to wish for anymore. There was a single feature however, that I really wanted to get into 5.0: identifying the name of the variable on which an assertion is executed.

Well, as of 5.0, when this assertion fails:

```csharp
IEnumerable numbers = new[] { 1, 2, 3 };
numbers.Should().HaveCount(4, "because we thought we put four items in the collection"))
```

The failure message will look like this:

> Expected numbers to contain 4 item(s) because we thought we put four items in the collection, but found 3.

Fluent Assertions will traverse the stack trace to find the line of code that invokes the assertion and then extracts the name of the variable or constant from your real C# files. So you'll need to build your unit tests in debug mode, even from a build server, to really benefit from this. In release builds, the compiler tends to inline lambda invocations. If it can't find this information, it will fall back on a more generic name like collection or object.

Note that analyzing the thread's stack trace is not supported in any .NET Standard preceding 2.0. So this feature will only work under .NET Standard 2.0 or the full .NET Framework. Also, if you've been building your own extensions around existing calls to `Should()`, consider tacking on the `[CustomAssertion]` attribute. You can read more about this in the @extensibility guidelines.

#### Redefining equivalency

Since I was on a roll to introduce breaking changes anyway, this release finally gave me the opportunity to repair quite a few of the behavioral design mistakes in the structural equivalency API. So in addition to the aforementioned change from `ShouldBeEquivalenTo` to `Should().BeEquivalentTo`, a lot more has changed.

First of all, the equivalency algorithm will now use the expectation to drive the comparison. For years, it would use the properties and/or fields of the subject-under-test to run a recursive comparison. Don't ask me why, because I don't remember that anymore (or I have blocked this part of my brain). But now, the expectation that you pass in really represents what you expect the subject to look like. This also make it very natural to use an anonymous type as the expectation.

Another thing I've changed is to disable auto-conversion of member values. This has been requested for many times, mostly because it confused so many people. For example, the conversion logic would allow you to treat a `DateTime` property and its string-representation as equivalent. This is no longer happening, but if you really want to, you can still opt-in this option using the `WithAutoConversion` and `WithAutoConversionFor` methods.

Similarly to auto-conversion, I've considerably simplified the way Fluent Assertions determines whether or not an object has value semantics. Before this release, it used a static `IsValueType` lambda and some awkwardly unclear heuristics. As of now, any type that overrides `Object.Equals` is treated as having value semantics. Why? Well, the entire purpose of that method (and its sibling `GetHashCode`) is to allow you to add value semantics to a reference type. Why wouldn't I comply to that .NET design principle? However, I also acknowledge the fact that not everybody will follow this principal faithfully, so you can override this using the `ComparingMyMembers` and `ComparingByValue`. Don't worry, I have some upgrading tips at the bottom of this post. Oh, and don't forget you can also set these and options globally using `AssertionOptions.AssertEquivalencyUsing`. Read all about this in the [updated documentation](http://fluentassertions.com/documentation.html).

#### Formatting your objects beautifully

The formatting engine in Fluent Assertions is based on built-in and custom implementations of the `IValueFormatter` interface. Unfortunately, this design has suffered from a long-standing design mistake. It could not properly detect cyclic references. The fix for that required me to change the method signature in a breaking way:

```csharp
string Format(object value, FormattingContext context, FormatChild formatChild);
```

The context parameter provides information about the depth of the graph as well as an indicating whether the formatter should use line-breaks in its output. But the fundamental change here is the `FormatChild` delegate that is passed in. In previous releases, if a formatter needed to format data itself, it would directly call `Formatter.ToString`. But that did not allow me to keep track of the graph that was being formatted. By using the `formatChild` parameter instead, Fluent Assertions will automatically detect a cyclic dependency and display a clear message for that value. If you want to build your own formatters, check out the [extensibility guide](http://fluentassertions.com/extensibility.html).

#### New event monitoring API

Being able to assert that a C# event was raised has been part of the API for years now. But with the trend of multi-threading development and the introduction of async and await, this API started to fall apart. It relied on thread-static state (did I already mention how bad static mutable state is?). So in this release, I've introduced a slightly modified syntax that makes the monitoring scope explicit and independent of the thread on which something is running.

```csharp
var subject = new EditCustomerViewModel();

using (var monitoredSubject = subject.Monitor())
{
  subject.Foo();
  monitoredSubject.Should().Raise("NameChangedEvent");
}
```

Note that the object you execute the `Should().Raise` call on is not the same object as your subject. The Monitor method returns an object implementing `IMonitor` as an override of `IDisposable` that defines when monitoring should be stopped. And for those people that love to build their own assertion, that object exposes a load of metadata that you can use any way you can. If you want to learn more about this, check out the [updated documentation](http://fluentassertions.com/documentation.html).

#### Upgrading tips

So while dogfooding the betas and release candidates on our own projects, I collected a couple of notes that might help you understand any issues that you may run into while upgrading. In general, be prepared for discovering some false-positives that were hidden because of earlier bugs in Fluent Assertions. 

The changes to `BeEquivalentTo` will be the most visible ones:

* Disabling auto-conversion may cause some tests to fail because different types used to be convertible. Fix the expectation or use the `WithAutoConversionFor` option.
* Your tests may fail because of `BeEquivalentTo` reporting missing properties. This is caused by the expectation object being the driving factor for the structural comparison. Use `Including` or `Excluding` to fix that.
* They may also fail because the expectation doesn't define any properties. This is often a signal that you pass in an abstract type as the expectation. Change the expectation or use the `IncludeAllRuntimeProperties` option.
* Use `WithTracing` to understand how FA has evaluated your object graph.

Additionally:

* The date and time extensions such as those to define `20.September(2018).At(19, 51)` have moved to `FluentAssertions.Extensions`, so do a global regex text replace from:

```csharp
using FluentAssertions;
``` 

to

```csharp
using FluentAssertions;
using FluentAssertions.Extensions;
```

* `WithInnerException` returns the inner exception, so we removed `WithInnerMessage`. Just use `WithMessage` instead.</li></ul>

#### Sponsor us

If you check out the <a href="https://github.com/fluentassertions/fluentassertions/releases/tag/5.0.0">release notes</a>, you'll see that this release is quite big. But I could not have pulled this off without help from the community. First of all, a big shoutout goes to <a href="https://github.com/adamvoss">Adam Voss</a>, <a href="https://github.com/jnyrup">Jonas Nyrup</a> and <a href="https://github.com/krajek">Artur Krajewski</a> for helping me out finalizing this release. Next to that, I'm really thankful for the new logo provided by <a href="https://github.com/IUsername">Ben Palmer</a>. And finally, big thanks to <a href="https://www.jetbrains.com/">Jetbrains</a> for providing us with licenses for their new IDE, <a href="http://jetbrains.com/rider/download/">Rider</a>, as well as ReSharper. I honestly have not touched Visual Studio since I switched to Rider at the start of this project. 

And we need your help as well. Support us by becoming a sponsor at <a href="https://www.patreon.com/dennisdoomen">Patreon</a> or provide us with a one-time donation through <a href="https://www.paypal.com/us/cgi-bin/webscr?cmd=_flow&amp;SESSION=yMgzH0CK0Ym93cE-t-Rih4X7oubt81HNVywiN8gdixfP1CqCj1R1Vs3b_10&amp;dispatch=5885d80a13c0db1f8e263663d3faee8dc3f308debf7330dd8d0b0a9f21afd7d3&amp;rapidsState=Donation__DonationFlow___StateDonationLogin&amp;rapidsStateSignature=3c114d4cb78f27b4a9044dd076b6727af3341fec">Paypal</a>.

#### Help wanted

But now that version 5.0 is out of the door, don't think that the work is done. There's still a lot of feature requests. More than enough to keep a lot of contributors busy for the foreseeable feature. Just checkout the Github items marked with <a href="https://github.com/fluentassertions/fluentassertions/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22">Help Wanted</a> to get you going. Oh, and follow me at <a href="https://twitter.com/ddoomen">@ddoomen</a> to get regular updates on my everlasting quest for knowledge that significantly improves the way you build your projections in an Event Sourced world.