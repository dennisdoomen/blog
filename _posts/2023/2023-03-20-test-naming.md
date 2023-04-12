---

title: "How I keep my test names short and functional"
excerpt: "Naming in software is hard, so here's how I name and group my automated tests to use them as documentation"

tags:
- naming
- unit-testing
- test-driven-development
---

There are plenty of topics in software development land that cause passionate (and sometimes almost religious) debates. Tabs vs spaces is most definitely one of them. And in .NET land there's always some team debating about the use of `_` in private fields and `var` for local variables. Fortunately, naming unit tests isn't such a hot potato and a lot of variations are widely accepted. But I still care about having functional and self-explanatory names. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/naming-tests.png" class="align-center"/> 

I never really liked the idea that (I think) was introduced by Roy Osherove in his famous book [The Art of Unit Testing](https://www.artofunittesting.com/). He proposed to use a convention like `[UnitOfWork]_[StateUnderTest]_[ExpectedBehavior]`. This results in test names like `Sum_NegativeNumberAs1stParam_ExceptionThrown`. Although I totally dig the use of underscores, those names are way too technical and cryptic to me. As I wrote in an [earlier post]({% post_url 2016/2016-11-06-the-three-mental-modes-of-working-with %}), I see the name of the test as an important piece of information. For me, it should explain the functional scenario that the test is trying to assert and not a description of how the test behaves. 

For a long time I have preferred the more functional `When_[scenario]_it_should_[expected_behavior]`. For example `When_the_same_objects_are_expected_to_be_the_same_it_should_not_fail`. It's functionally correct, but I'm seeing this style more and more as noisy and verbose. So let's look at how we can make it shorter. Consider the tests below from [Fluent Assertions](https://fluentassertions.com/):

```csharp
public class ReferenceTypeAssertionsSpecs
{
    [Fact]
    public void When_the_same_objects_are_expected_to_be_the_same_it_should_not_fail()
    {
        // Arrange
        var subject = new ClassWithCustomEqualMethod(1);
        var referenceToSubject = subject;

        // Act / Assert
        subject.Should().BeSameAs(referenceToSubject);
    }
}
```

Given that test is part of the `ReferenceTypeAssertionsSpecs` class (and notice the `Specs` postfix I'm always using), you can safely assume that this is one of the test sets covering the fluent APIs that deal with reference types. In this particular example, it's covering the `BeSameAs` method, and that's just one of many. 

A common pattern I've been adopting lately is to group the test using a nested class. I'm essentially providing more context about a group of related tests. By doing that, I can immediately remove some superfluous information from the test name: the fact that we're expecting objects to be "the same". 

```csharp
public class ReferenceTypeAssertionsSpecs
{
    public class BeSameAs
    {
        [Fact]
        public void When_two_variables_are_referring_to_the_same_object_it_should_not_fail()
        {}
    }
}
```

Although the names read much more natural, given the context of `BeSameAs`, they are still quite long. So let's try some versions of that when we drop terms like _should_ and/or _when_:

1. `Should_succeed_when_the_variables_refer_to_the_same_object`
2. `Succeeds_when_the_variables_refer_to_the_same_object`
3. `The_variables_must_refer_to_the_same_object`
4. `Must_refer_to_the_same_object`

I think the fourth is a bit too short, so let's continue with option 3. But what if there's another test that tests the opposite and that throws if the two variables do not point to the same object? Technically, `The_variables_must_refer_to_the_same_object` would fit here as well, so we need some way to make the distinction between the happy and the unhappy paths. Let's try to capture the "fact" that we're really trying to assert here instead of describing the behavior of the test:

* `References_to_the_same_object_are_valid`
* `References_to_different_objects_are_invalid`

Pretty short right? And I think it's concise enough to still sound like proper English? Is it possible to make this shorter? Sure, I'm convinced I'll come up with something better in a couple of months. But it's good enough like this. Naming things still remains one of the hardest things in our profession. 

To wrap up this post, here's the final version:

```csharp
public class ReferenceTypeAssertionsSpecs
{
    public class BeSameAs
    {
        [Fact]
        public void References_to_the_same_object_are_valid()
        {}

        [Fact]
        public void References_to_different_objects_are_invalid()
        {}
    }
}
```

So what naming conventions do you use for your tests? And what do you think of what I'm proposing here? Let me know by commenting below. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).