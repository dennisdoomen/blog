---

title: "What's the \"unit\" in unit testing and why is it not a class"
excerpt: "If you choose the wrong unit testing scope, you'll regret adopting unit testing and TDD in the first place"

tags:
- unit-testing
- test-driven-development

---

## Why care about the scope of testing?
Somewhere in 2018, I asked my Twitter friends for advice on defining a heuristic to define the right scope of your unit tests. This resulted in some interesting discussions, but I still remember two responses that somehow stuck. I particularly liked the humor in the first one:

> When someone else can modify your code safely, without you getting sweaty armpits, the scope of your unit test is okay

The other one sounded more thoughtful and wise:

> Our unit test should be large enough that you can assert something meaningful, but small enough that you can quickly read & assess it

You may wonder why we should care about this in the first place. Well, I hope you do agree with the value of unit testing. In my experience, It can help produce code that can be changed by any developer in the team without fear and with confidence. But unit tests do not come for free. They can easily extend the _initial_ development time with 50%. But I promise you, your return of investment will be significant. You'll end up with happier developers and happier clients. 

But that's not what I meant with "free". The "dark side" of unit testing and Test Driven Development, as some like to call it, is that you can do it wrong. And if you do, it will hurt all _successive_ development in such a way that you regret adopting unit testing in the first place. Fortunately for you I've already shot myself in my feet extensively and thus have a lot of experience to share. This already let to my [recent post]({% post_url 2021/2021-10-21-laws-test-driven-development %}) on the "laws" of test driven development. But I never elaborated on how to find the right scope for automated testing. 

## A real-life example involving databases
Let's start with the first example. Consider a type which main purpose is to provide general database management operations. It has a method that will check a particular table exists, and if not, create it. 

```csharp
_databaseManager.EnsureTableExists("users");
```

Now ask yourself, what should be the scope of the automated tests?

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/uml-databasemanager.png" class="align-center" style="max-width: 500px"/> 

If you follow the guidance by some of the books on this topic, every type should be covered by a separate set of tests. So one set for the `DatabaseManager` and assuming the factory can be covered in one go, one set for the `SqlDatabaseAdapter`. However, I don’t think you can make that decision without understanding the relationship between those types. Are the adapter and its factory part of the same layer or module? Is any of them supposed to be reusable or are they just implementation details to make the code easier to change in the future? What were the original requirements that led to this design? 

After consulting with the developer that designed this, it turned out that there's really only one implementation of the `IDatabaseAdapter`. He added those interfaces just to "be ready for the future" or to "be SOLID". There was no requirement to support any other database than SQL Server, and as far as we know, there never will be. In fact, the existing `DatabaseManager` tests were creating a mock of the `IDatabaseAdapterFactory` that returns a mock of the `IDatabaseAdapter`. This is the result of he manager delegating all the "dirty" interaction with SQL Server to the adapter. In other words, those tests were only ensuring that a call to `EnsureTableExists` resulted in a call to `IDatabaseAdapter.EnsureTableExists`. The actual adapter wasn't covered at all. Since the primary purpose of that manager is to interact with the database, testing only the mocks is quite wasteful. So for these _specific tests_ I would just [use a Linux test container]({% post_url 2023/2023-03-05-docker-in-tests %}) running SQL Server to cover everything the `DatabaseManager` is supposed to do. 

In my opinion, the original developer didn't understand the subtleties behind SOLID and applied the guidelines rather dogmatically. Given the requirements at that time, it could have all been a single class. Only when there would be a need to support multiple database vendors, I would have considered refactoring and introducing the Adapter pattern. And that's my point. Even if those abstractions _were_ needed at some point, they would be the result of refactoring. The original purpose of the `DatabaseManager` wouldn't change. And you shouldn't need to rewrite your tests if you decide to refactor the implementation from a single class into multiple classes. Refactoring shouldn't change the purpose, nor the behavior. That's why testing too small is such a bad practice. It can complete kill your ability to move fast. 

## Another example from FluentAssertions
As you may know, [FluentAssertions](https://fluentassertions.com/) has a feature to compare two object graphs even if the types in those graphs differ. This capability, available through the `BeEquivalentTo` method, allow you to do something like this:

```csharp
eventMonitor.OccurredEvents.Should().BeEquivalentTo(new[]
{
    new
    {
        EventName = "PropertyChanged",
        TimestampUtc = utcNow - 1.Hours(),
        Parameters = new object[] { “third”, “first”, 123 }
    },
    new
    {
        EventName = "NonConventionalEvent",
        TimestampUtc = utcNow,
        Parameters = new object[] { "first", 123, "third" }
    }
}, o => o.WithStrictOrdering());

```

It executes a recursive comparison member by member. And it does that in a smart way. For instance, types that have members themselves and do not override `Equals` are compared by recursively traversing their members. Dictionaries are equivalent if they have the same keys and their values are equivalent (again by running a nested recursive comparison). And collections are equivalent when they contain the same equivalent object in any order (unless you use something like `WithStrictOrdering`). And it doesn't stop there. Here's a class diagram showing just a subset of the implementation.  

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/uml-fluentassertions.png" class="align-center"/> 

`BeEquivalentTo` is a method on a class that is returned from the `Should` extension method. When I started the implementation of this API almost ten years ago, there was only a single class to implement the behavior: `EquivalentValidator`. But over the years, I added more and more capabilities and needed to refactor the implementation to break it down into smaller and well-focused supporting class. And that's exactly what the original authors of the Design Patterns book meant when they said they should have named the book "Refactoring towards Design Patterns". And just like the previous example, refactoring my code shouldn't affect the behavior, the API, and more importantly, the tests. Applying the test-per-class strategy would have completely screwed up my ability to refactor.

## A less trivial example
Consider the following call:

```csharp
var user = await _httpClient.GetAsync("/api/users/1234", body);
```

In the .NET world, this is most likely implemented like this:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/uml-usermodule.png" class="align-center" style="max-width: 500px"/> 

Now what should the test scope be here? A developer who just started with TDD would probably write individual tests for the `UsersController` and the `SqlUserRepository`. But given what I've been trying to tell you in this post, I guess your default answer would be to test the controller and repository in one go. 

Well, I think the correct answer is "it depends". Are the controller and repository part of the same module or functional slice and specifically built for that? If so, I would most likely cover both in one set of tests (possibly using a SQL Server docker container). But if this is part of some kind of Onion or Hexagon Architecture (and thus using the Dependency Inversion Principle), it is very likely that the `IUserRepository` is a specific interface owned by the same module that owns the `UsersController`. Other modules may have their own version of that interface. In that case, the `SqlUserRepository` is implementing those interfaces, and by definition, lives outside the scope of the controller. And because of this, I would most definitely test the repository separately.

With these three real-world examples, I hope you see my point that the default test-per-class idea is rubbish. But if that's the case, how do you find the right scope then? Unfortunately there aren't simple rules and guidelines to determine that scope. It really depends on the architecture and the internal boundaries of your code base. But in my next post, I'll give you some heuristics and smells that will help identify the right boundaries. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).
