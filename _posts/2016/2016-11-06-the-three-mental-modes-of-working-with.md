---
title: The three mental modes of working with unit tests
date: '2016-11-06T20:22:00.001+01:00'
tags:
- unit testing
- test driven development
modified_time: '2016-11-06T20:22:23.334+01:00'

---

The other day, while pairing up on some unit test, I started to realize that I generally have three modes of looking at my unit tests.

**The Writing Mode**

While writing, I mostly focus on the mechanics of getting the test to pass. By then, I usually have a mental model and a particular scenario in mind, and my thoughts mostly focus on finding the most elegant syntax and structure to get my test from red to green. Since I already know the exact scenario, I don't put too much attention on the name. If I'm really into the flow, the edge cases and alternative scenarios just pop into the back of my mind without me needing to really think about. In this mode, I also spend a lot of thoughts to come up with opportunities to refactor the test itself or the underlying constructs. For instance, is the scope of my test correct, does the subject-under-test not have too many dependencies. Since I practice Test Driven Development, some of these refactoring opportunities surface quick enough when I my set-up code explodes, or when my test code doesn't communicate the intent anymore.

**The Review Mode**

While reviewing somebody's pull request I switch to review mode in which I use the unit tests to understand the scope, the responsibilities and the dependencies of a class or set of classes. To understand those responsibilities, I put particular attention to the names of the tests thereby completely ignoring the implementation of the test itself. With the names as my only truth, I try to understand the observable behavior of the subject-under-test (SUT) under different scenarios. They should make me wonder about possible alternative scenarios or certain edge cases. In other words, they should make it possible for me to look at the code from a functional perspective. That doesn't mean they need to be understandable by business analysts or product owners, but they must help me understand the bigger picture.

Only when I'm satisfied that the developer considered all the possible scenarios, I start to look at the implementation details of particular test cases. What dependencies does the SUT have? Are there any I didn't expect? If so, did I understand the test case correctly, or is the test hiding important details? Are all dependencies I did expect there? If not, where are they? Is everything I see important to understand the test? If not, what aspects could be moved to a base-class (for BDD-style tests), or is a [Test Data Builder](http://www.natpryce.com/articles/000714.html) or [Object Mother](https://github.com/dennisdoomen/LiquidProjections/blob/TddDemo/Tests/LiquidProjections.RavenDB.Specs/12_ObjectMothers/RavenProjectorSpecs.cs#L63) a better solution? Do all assertion statements make sense? Did he or she use any constant values that are difficult to reason about? Is each test case testing a single thing. What if the test fails? Does it give a proper message to the developer what went wrong functionally or technically? A proper [assertion framework](http://www.fluentassertions.com/) can help because what use would the error "Expected true, but found false" have?

**The Analysis Mode**

Now, consider a test fails and I'm the one that needs to analyze the cause of this. In this debugging mode, I first need to understand what this test was supposed to verify. For this, I need a name that clearly explains the specifics of the test case on a functional level. Again, I won't let my thoughts be distracted by the implementation. The name should help me understand what is the expected behavior and help me make up my mind on whether that scenario makes sense at all. After I conclude that the test case indeed makes sense, I'll start studying the implementation to determine if the code really does do what the test name suggests. Does it bring the context in the right state? Does it set up the dependencies correctly (either explicitly or through some kind of mocking framework)? Does it invoke the SUT using the right parameters? And does the assertion code expect something that makes sense to me considering the initial state and the action performed? Only if I've confirmed the correct implementation, it's time to launch a debugger.

I know the world is not perfect, but keeping out of the debugger hell should be a primary concern for the test writer. This is a difficult endeavor and requires the developer to ensure the intent of a unit test is as clear as crystal. Naming conventions, hiding the irrelevant stuff, and a clear cause and effect are adamant to prevent yourself from shooting in your own foot in the long run. If you're looking for tips to help you in this, consider reading my prior post on writing maintainable unit tests.

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).

