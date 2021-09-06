---
title: "12 tips to write unit tests that don't cripple your codebase"
tags:
- test driven development
- unit testing
---

Over the last months, I've been involved in more and more code reviews, mostly because we've increased the level of quality required for code changes to our code base. While doing that, I started to track my most frequently used review comments intended to improve. 

1. If the dependencies of your subject are not important for a particular test, use a [Test Data Builder](http://natpryce.com/articles/000714.html) to build the subject-under-test (SUT). However, always have at least one test that clearly show those dependencies. Hiding them increases the change your subject ends up with more dependencies then what's good for you.

2. Don't repeat the action your executing against the SUT in the name of the class. For instance, tests for a query handler should not start with `When_executing_…`. That's superfluous information that does not help to keep the purpose of the test apart.

1. Be very conservative in applying the typical [Don't Repeat Yourself](http://c2.com/cgi/wiki?DontRepeatYourself) (DRY) principles to unit tests. They tend to obscure the cause and effect, especially if you're moving stuff into some kind of base-class (or worse, a hierarchy of base-classes).

1. Don't introduce constant variables and such at the top of the test, unless you're dealing with a magic number. For example, I often use literals like `"TheIdentifier"` and `"OtherIdentifier"` to set them apart and communicate intend. Even numeric values like `1123` don't need constants, even if you use them multiple times. It's quite obvious not just a number. But, numbers like `1`, `0` and such, don't say much, and usually benefit from a well-named constant.

1. Hide everything that is not important to see in that particular test. In other words, if it's not important for the test, it's very important NOT to show it. Test Data Builders or a simple [With](https://github.com/Erwinvandervalk/Chill/blob/develop/Source/Core/Chill.Shared/ObjectExtensions.cs) extension method can help hide the unimportant. But again, be careful to hide design errors by adding intelligence into the test data builders. That's the software equivalence of sweeping the dirt under the  carpet.

1. Ensure that the cause and effect of the test is crystal clear. Nothing is more annoying than a failing test that makes no sense at all, neither by looking at the name, nor the implementation. If you're practicing Test Driven Development well, the tests can serve as documentation for your API. So make sure your tests read like a book.

1. If the test fails, ensure that the failure message is so clear that no debugging sessions are necessary. Using a proper [assertion library](https://www.fluentassertions.com) is not a luxury anymore. One note though. If you love [FakeItEasy](https://fakeiteasy.github.io) like I do, don't use constructions like this

    ```csharp
    A.CallTo(() => commandService.Execute(A.That.Matches(cmd =>
        (cmd.Site == new SiteName(equipment.Owner.ToString())) &
        (cmd.Identity == equipment.Code))))
        .MustHaveHappened(); 
    ```

    If the `Site` and `Identity` properties of the command contain anything but the expected values, you'll need a debugger to figure out what happened.

1. Don't stick to a single style. I generally make a distinction between tests that do simple state testing and those that are more orchastration oriented. For the former, I prefer the [AAA syntax](https://github.com/fluentassertions/fluentassertions/blob/master/Tests/Shared.Specs/FormatterSpecs.cs#L17), whereas the latter is more suitable for a BDD-style test, e.g. using [Chill](https://github.com/ChillBDD/Chill). But be consistent. Don't mix more than two styles in a single code-base (or git repository).

1. Whether to test small or test big is a topic that is a source for heated debates. I see value in both of them, so depending on the thing I'm building you might see me start of with an end-to-end integration test and then drill down into the code base to add more fine grained unit tests. Reverting this process is perfectly fine as well. However, if you don't start off with such an end-to-end test, you might discover that you didn't design for it, making it diffuclt to add it later on.

1. If you need to verify that a certain string value or exception message matches the expectation, never verify the exact message. Use wildcards to verify the specific parts that are relevant to verify that the behavior meets the expectation. It will save from unexpectedly failing tests if you decide to refine the text involved.

1. Don't refer to technical names such as the names of methods, variables and UI elements in the name of a test. In my opinion, the name of a test should explain the functional scenario, whereas the cause and effect should clearly illustrate the technicalities of that scenario. If some class or member is renamed, it shouldn't affect the functional name of the test.

1. Be careful with libraries like [AutoFixture](https://github.com/AutoFixture/AutoFixture). They have value, but using those for everything increases the chance your test starts to fail on something that is irrelevant for that specific test. Moreover, they usually have a negative impact on performance. And again, if the value of certain object is important to understand the test scenario, it is VERY important to show that.

1. As usual, these opinions are mine, so feel free to comment below. Oh and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.