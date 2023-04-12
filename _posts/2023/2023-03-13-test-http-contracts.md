---

title: "How to properly test your HTTP API contracts in .NET"
excerpt: "The route, the headers and the specific JSON returned by an HTTP API are the contract, and thus should be treated as such"

tags:
- api
- unit-testing
- test-driven-development

---

As I'm a Test Driven Development practitioner (with conviction), I regularly have to create automated tests that include the HTTP API of a module or component. Whether you call that a unit, integration or component test is debatable, but is beyond the point of this post. What I do care about that is that your tests only interact with the surface area designed for your production code. So if a particular part of your system is only invoked through an HTTP API, then your test should be doing the same thing. Directly invoking a method on an ASP.NET `Controller` class would violate that idea. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/api-testing.png" class="align-center"/> 

[Another important principle]({% post_url 2021/2021-10-21-laws-test-driven-development %}) which I follow in my testing endeavors is to ensure you only assert what is relevant for that particular test case. If you expect an exception, ensure it is the right type of exception, that its properties have the right value, and that its message matches your expectation. But with respect to the exception message, you only want to assert the relevant parts of that message. [Fluent Assertions](https://fluentassertions.com/)' `WithMessage` assertion takes a wildcard for that exact reason. Similarly, if your API is supposed to return a particular HTTP error code, only assert that it does and ignore the payload. If your test covers a particular path where only a specific property of the body is relevant, ignore the rest. This avoids failing tests for unrelated issues. 

I've seen a lot of developers that reuse the JSON-serializable type from the production code in the test code to deserialize from. A common argument developers give for that is that it makes the code more refactoring-friendly. In other words, changing the name of a property on that type (often used as a Data Transfer Object) would not break the test. But in my opinion, that _should_ break the test. The route, the headers and the specific JSON returned by an HTTP API _are_ the contract, and thus should be treated as such. 

But how do you do that? There are two common ways: using raw JSON or by deserializing the body to an anonymous type of a particular structure. Using raw JSON is the most pure and thorough way of doing that, but will become ugly when you only want to assert the relevant parts match the expectation. 

Deserializing to an anonymous type can be done like this when you use `NewtonSoft.Json` and Fluent Assertions:

```csharp
IHost host = GetTestClient();

HttpResponseMessage response = await host.GetAsync(
    $"http://localhost/statistics/metrics/CountsPerState?country={countryCode}&kind=Filming");

string body = await response.Content.ReadAsStringAsync();

var expectation = new[]
{
    new
    {
        State = "Active",
        Count = 1
    }
}

T actual = JsonConvert.DeserializeAnonymousType(body, expectation);

actual.Should().BeEquivalentTo(expectation);
```

What we're doing here is to set-up the `expectation` with specific values and then using `DeserializeAnonymousType` to tell `NewtowSoft.Json` to try to deserialize the JSON into an anonymous object which structure is defined by that same `expectation` object. We complete the test by using `BeEquivalentTo` to do a deep comparison between `expectation` and `actual`, where the `expectation` defines the properties we care about. 

If you prefer `System.Text.Json`, we can do achieve the same result like this:

```csharp
IHost host = GetTestClient();

HttpResponseMessage response = await host.GetAsync(
    $"http://localhost/statistics/metrics/CountsPerState?country={countryCode}&kind=Filming");

string body = await response.Content.ReadAsStringAsync();

var expectation = new[]
{
    new
    {
        State = "Active",
        Count = 1
    }
}

object actual = JsonSerializer.Deserialize(body, expectation.GetType(), new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
});

actual.Should().BeEquivalentTo(expectation);
```

You could encapsulate most of this logic into a custom `BeEquivalentTo` that acts on a `HttpResponseMessage` like I did [here](https://github.com/dennisdoomen/EffectiveTddDemo/blob/master/Tests/DocumentManagement.Specs/13_SimplerDeserialization_NewtonSoft/HttpClientExtensions.cs#L8) for `NewtonSoft.Json` and [here](https://github.com/dennisdoomen/EffectiveTddDemo/blob/master/Tests/DocumentManagement.Specs/14_SimplerDeserialization_SystemText/HttpClientExtensions.cs#L8) for `System.Text.Json`. But you could also start using the `Should().BeAs()` provided by this community library called [FluentAssertions.Web](https://github.com/adrianiftode/FluentAssertions.Web#fluentassertionsweb-examples). 

What do you think about this approach? Do agree with the principles? And do you test your HTTP APIs like this too? Let me know by commenting below. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).