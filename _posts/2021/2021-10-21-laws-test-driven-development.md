---

title: "Dennis' 17 Laws of Test Driven Development"
excerpt: "These are the 'things' that have helped me avoid shooting myself in my own foot while practicing Test Driven Development."

tags:
- test driven development
- testability
---

About 14 years ago, I got inspired by a series of posts written by (Alt) .NET veteran [Jeremy D. Miller](https://jeremydmiller.com/)]. Now, with many years of both good and bad experiences practicing Test Driven Development, I thought it is the time to honor Jeremy by capturing my own "laws". It goes without saying that using the term _law_ is an exaggeration and _principles_ or _heuristics_ cover my intend much better. 

Nonetheless, these are the "things" that have helped me avoid shooting myself in my own foot.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2021/laws.jpg" class="align-center"/> 

1. Don't practice Test Driven Development according to the books. Sure it's a great technique to design testable software and control your dependencies. Yes, it forces you to think about the behavior of the subject-under-test. And indeed, it can lead to tests-as-documentation of your domain. This does not mean, however, that you should always start with a test. Especially when you’re designing new code, often, you first need to sketch out the class responsibilities, flesh out the details and then refactor the code so you can continue with a test-first approach.

1. Don't bother trying to agree on an exact definition of what the difference is between unit or integration tests. Just accept that you need tests on different levels (e.g. class, component, module, API, UI) that are complimentary. Arguing about which one is better is a non-sensical waste of time.

1. Choose the right scope for your tests. I'll dedicate a separate post on this, but suffice to say that a test-per-class is almost always the wrong approach. Look at which classes are supposed to work together, which are the ones used directly by other parts of the system, and which are just implementation details. This particularly works well if you organize your code in functional folders. 

1. Start with a functional name that describes what scenario the test case is supposed to validate. Most definitely don't include the literal names of classes, types, methods or even the names of buttons in the UI. This helps the next developer to judge whether the test implementation actually does what it is supposed to do when it fails. 

1. Use naming conventions like `When_something_happens_it_should_do_something`, `Should_do_something_when_something_happens` or, even more concise `Do_something_when_something_happens`, but be consistent within a code base. Although JEST and Jasmine tests need a slightly different structure, the point of being functional remains. 

1. Strive for a crystal-clear cause and effect, where it's completely clear what the begin state is, what the action or subject-under-test is, and what you are asserting in this specific test. In other words, if you need to browse through all the helper methods or base-classes to understand the test, there's definitely room for improvement. 

1. Only have one scenario per test. So doing something, asserting the outcome, then doing something else and then _again_ asserting the outcome should be avoided. It makes the test harder to read, and will make it skip the second or third scenario if the first one fails. 

1. Treat test code as first-class citizens of your code-base. In other words, everything that applies to your production code, also applies to your tests. Review them, refactor them, and evolve them, just like all the other code.

1. But be cautious with DRY (Don't Repeat Yourself) such as introducing base-classes for your tests or fixtures like the [Shared Context](https://xunit.net/docs/shared-context) offered by xUnit. It usually leads to hiding the important parts of the test case or too much intelligence in those base-classes or fixtures, resulting in the need to add virtual members so test cases can override that logic. 

1. Hide the set-up code that is not important, but show what's relevant for that specific test. Why am I saying this? Because each test case is supposed to assert something else and should fail for one reason only. Use [Object Mothers](https://martinfowler.com/bliki/ObjectMother.html) or [Test Data Builders](http://www.natpryce.com/articles/000714.html) to encapsulate the logic for building valid objects. 

1. As a consequence of the above, only assert what's relevant. For example, use [wildcards](https://fluentassertions.com/exceptions/#:~:text=WithMessage%20to%20support-,wildcard,-expressions%20and%20match) to check only the part of the exception message that you need to assert that the right exception was thrown. And when verifying objects in your tests, only assert on the properties or fields you care about in that specific test, for example, by comparing them with an [anonymous type](https://github.com/fluentassertions/fluentassertions/blob/master/Tests/FluentAssertions.Specs/Equivalency/BasicEquivalencySpecs.cs#:~:text=When_specific_properties_have_been_specified_it_should_ignore_the_other_properties).

1. Use inline literal strings and numbers instead of defining constants at the top of your test. Usually we want to do the opposite to avoid [magic numbers](https://csharpcodingguidelines.com//maintainability-guidelines/#AV1515), but in tests, having those constants often make the cause and effect harder to understand. 

1. Use a test framework that will run your tests in parallel to help you surface concurrency issues and race conditions as early as possible. 

1. Only test against the observable behavior of a class, module or component. In other words, avoid making "private" members "public", or writing directly to the database from your test. If you really need to access "protected" members, consider using something like a [test-specific subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html). But if you can, interact with the subject-under-test in exactly the same way as you would do in the production code.

1. Postfix your test classes with `Specs`. Doing this emphasizes the idea of writing tests as a specification of the API surface and the behavior of the subject-under-test.

1. Avoid overuse of mocking libraries. If your tests rely a lot on mocking libraries, or worse, use multiple mocks in your tests, chances are that you are either testing too small or your code-under-test has too many responsibilities. And never mock the subject-under-test itself. Then you know for sure that there’s something wrong with the responsibilities of that class.

1. Acknowledge that some tests are more state-based and others more orchestrational. The former fits well with the Arrange-Act-Assert conventions like this:

    ```csharp
    public void When_something_happens_it_should_do_something_else()
    {
        // Arrange
        ...setup the subject-under-tests and/or its dependencies
        
        // Act
        ...where we do the real work
        
        // Assert
        ...where we ensure the SUT has done what it should do
    }
    ```

    Orchestrational tests usually have multiple outcomes, so a more BDD-style fits much better. For example, using .NET's [Chill BDD library](https://chillbdd.com/) allow you to write them like this:

    ```csharp
    public class When_retrieving_existing_customer : GivenSubject<CustomerController, View> 
    {
        const int customerId = 12;
        
        public When_retrieving_existing_customer()
        {
            Given(() => {
                SetThe<Customer>().To(EntityMother.CreateACustomer().With(x => x.I = customerId));
            
                The<ICustomerStore>().GetCustomer(customerId).Returns(The<Customer>());
            });
            
            When(() => Subject.Get(customerId));
        }
    
        [Fact]
        public void Then_view_is_returned()
        {
            Result.Should().NotBeNull();
        }
        
        [Fact]
        public void Then_model_is_the_existing_customer()
        {
            Result.Model.Should().Be(The<Customer>());
        }
    }
    ```

So what do you think? Which of these practices have you already been using? Did you learn anything new? And did I miss something? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions
