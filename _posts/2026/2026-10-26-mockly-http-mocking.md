---
title: "Mockly: fluent HTTP mocking for .NET like it should have been done"
excerpt: "What Mockly does, how it makes testing code that uses HttpClient easier, and the features that help you test the unhappy paths as well."
tags: [.NET, Unit Testing, Test Driven Development, Open Source, Tooling]
---

## Why yet another mocking library?

If you ever tried to write a unit test for a class that uses `HttpClient`, you know what I'm talking about. `HttpClient` doesn't have an interface, so you can't just mock it. What most people end up doing is writing their own subclass of `HttpMessageHandler`, or using Moq or NSubstitute to override the protected `SendAsync` method using some magic strings. It works, but the result is a test where half of the code is plumbing. And when the test fails, you usually get a `NullReferenceException` or a 404 from somewhere deep in your code, without any clue what request was actually sent.

In my [earlier post](https://www.dennisdoomen.com/2023/04/unit-testing-scope.html) about the scope of unit tests, I explained that I don't like to mock every class. I rather test a larger piece of behavior and only replace the things that are really at the boundary of the system. Well, an HTTP call to another service is exactly such a boundary. So if I have to mock something, I want that mock to be trivial to set up, easy to read, and very explicit when a test fails. I couldn't find a library that did all of that in the way I wanted, so I built [Mockly](https://mockly.org).

## Getting started

Mockly comes as a core package and a separate package with the assertions. Which assertion package you need depends on the version of Fluent Assertions you use.

```bash
dotnet add package mockly
dotnet add package FluentAssertions.Mockly.v8   # or FluentAssertions.Mockly.v7
```

It runs on .NET Framework 4.7.2 and higher and on .NET 8.0 and higher. This is what a first test looks like:

```csharp
var mock = new HttpMock();
mock.ForGet()
    .WithPath("/api/users/123")
    .RespondsWithJsonContent(new { Id = 123, Name = "John Doe" });

HttpClient client = mock.GetClient();

var response = await client.GetAsync("/api/users/123");
var content = await response.Content.ReadAsStringAsync();

response.StatusCode.Should().Be(HttpStatusCode.OK);
content.Should().Contain("John Doe");
mock.Should().HaveAllRequestsCalled();
```

No configuration files, no base classes. You create an `HttpMock`, tell it what requests to expect and what to return, and ask it for a client. That client already has its `BaseAddress` set to `https://localhost/`, so relative URLs just work.

Of course, not all code takes an `HttpClient` in its constructor. That's why you can also ask the mock for an `IHttpClientFactory` using `GetClientFactory()`, which is what you need if your code gets its clients through dependency injection. And if you build the `HttpClient` yourself or have some custom infrastructure, `GetMessageHandler()` gives you the raw `HttpMessageHandler`.

## Telling Mockly what to expect

Every mock starts with the HTTP method. So you have `ForGet()`, `ForPost()`, `ForPut()`, `ForPatch()`, `ForDelete()`, `ForHead()` and `ForOptions()`. And if you happen to talk to something exotic like WebDAV, you can use `For(new HttpMethod("PROPFIND"))`.

After that, you describe the request as detailed as you need. You can configure the scheme, host, path and query separately, but I find it often easier to just pass a full URL to the `ForXxx` method. You can use wildcards in the host, the path and the query.

```csharp
mock.ForGet("https://api.example.com/users/*?q=*")
    .RespondsWithStatus(HttpStatusCode.OK);

mock.ForPost("http://*.example.com/*")
    .RespondsWithStatus(HttpStatusCode.Created);
```

Paths can also contain named placeholders like `{id}`. I'll come back to those later, because they become really useful when you want to build a response from the request.

Query strings got a lot of attention. `WithQuery("?q=*&limit=10")` matches the full query string, and `WithAnyQuery()` and `WithoutQuery()` do exactly what their names say. But in practice, you often don't control the order of the query parameters. That's what `WithQueryParam` is for. It matches a single parameter no matter where it appears, and if you don't pass a value, it only checks that the parameter is there.

```csharp
mock.ForGet()
    .WithPath("/api/search")
    .WithQueryParam("q", "mockly*")
    .WithQueryParam("page", "1")
    .WithQueryParam("api-key")
    .RespondsWithStatus(HttpStatusCode.OK);
```

Headers have their own methods as well. `WithHeader("X-Api-Key")` only checks if the header exists, while `WithHeader("X-Trace-Id", "abc-*")` also checks the value. For multi-valued headers, it's enough if one of the values matches. Then there's `WithBearerToken("eyJ*")` for the `Authorization` header and `WithContentType("application/json")`, which ignores things like the `charset` parameter. And if you need to mock an OAuth token endpoint, `WithFormField("grant_type", "client_credentials")` will look inside a URL-encoded form body.

### Matching the body

For a `POST`, `PUT` or `PATCH`, the body is usually the most important part. You can match it with a wildcard pattern using `WithBody("*something*")`, compare it as JSON with `WithBodyMatchingJson(...)` or use a regular expression through `WithBodyMatchingRegex(...)`. But my personal favorite is to pass an object:

```csharp
mock.ForPatch()
    .WithPath("/api/relationships/42")
    .WithBody(new
    {
        EntityKey = "TheRuleKey",
        RepresentativeId = "abc123"
    })
    .RespondsWithStatus(HttpStatusCode.NoContent);
```

Mockly will serialize that object and compare it with the request body as JSON, so whitespace and layout don't matter. If your API uses camel casing or other serialization settings, just add `Using(options)` with your own `JsonSerializerOptions`. Mockly will then use those for both matching and responding.

One thing to be aware of. Body matching only works if Mockly recognizes the content type as text, like `text/*`, `application/json` or `application/xml`. If you use some custom content type that is actually text, you can tell Mockly by calling `TreatBodyAsTextual()`.

And if none of these matchers do what you need, you can always pass your own predicate to `With()`. It can even be `async`.

```csharp
mock.ForPost()
    .WithPath("/api/test")
    .With(req => req.Body!.Contains("something"))
    .RespondsWithStatus(HttpStatusCode.NoContent);
```

### Just paste a cURL command

This is a small feature, but it saves me a lot of typing. Most browsers allow you to copy a request as a `curl` command from the developer tools. You can paste that directly into Mockly:

```csharp
mock.ImportFromCurl("""
    curl -X POST https://api.example.com/users \
      -H 'Content-Type: application/json' \
      --data-raw '{"name":"mockly"}'
    """)
    .RespondsWithStatus(HttpStatusCode.Created);
```

Mockly will turn the method, URL, headers and body into the same setup you would otherwise have written by hand. If there's no `-X`, it assumes `POST` when there's a body and `GET` when there isn't.

## What to respond with

The obvious ones are `RespondsWithStatus`, `RespondsWithContent("Hello", "text/plain")`, `RespondsWithEmptyContent(HttpStatusCode.NoContent)` and `RespondsWithJsonContent`. You can add response headers like `Location` or `ETag` with `WithHeader`. You don't have to remember which headers belong on the content and which ones on the response. Mockly will put them in the right place for you.

```csharp
mock.ForPost()
    .WithPath("/api/users")
    .RespondsWithStatus(HttpStatusCode.Created)
    .WithHeader("Location", "/api/users/123")
    .WithHeader("ETag", "\"v1\"");
```

Then there are a couple of response types that I kept needing in real projects. Many modern APIs return errors as [RFC 7807 Problem Details](https://datatracker.ietf.org/doc/html/rfc7807), so there's `RespondsWithProblemDetails` which takes the status code, title, detail, type, instance and any extensions you want. For downloads, you can use `RespondsWithFile("report.pdf")`, which opens the file again for every request and guesses the content type from the extension. `RespondsWithBytes` is safe to call multiple times as well, but `RespondsWithStream` isn't, simply because a stream can only be read once. If you work with OData, `RespondsWithODataResult` will wrap your data in an OData envelope. And for anything else, like a multipart batch response, you can pass any `HttpContent` to `RespondsWith`.

### Building the response from the request

A fixed response is fine for most tests. But sometimes you need the response to depend on what was sent. For that, the `RequestInfo` object that Mockly passes around has a couple of helper methods:

```csharp
req.BodyAs<User>();      // deserializes the body, honoring Using(options)
req.PathSegment(2);      // "/api/users/123" -> "123"
req.QueryValue("page");  // the value of the "page" query parameter, or null
req.RouteValue("id");    // the value of a {id} placeholder in the path, or null
```

Combine that with the route placeholders I mentioned before, and you can build a small fake API in just a few lines:

```csharp
mock.ForGet("/api/users/{id}")
    .RespondsWithJsonContent(req => new { Id = req.RouteValue("id"), Status = "active" });

mock.ForPost("/api/users")
    .RespondsWithJsonContent(HttpStatusCode.Created,
        req => new { Id = Guid.NewGuid(), Name = req.BodyAs<User>().Name });
```

If you want full control, `RespondsWith` also accepts a delegate that creates the entire `HttpResponseMessage`. That delegate can be `async` and can take the `CancellationToken` of the HTTP pipeline, so you can pass it on to whatever you're awaiting.

### Reusing your test data builders

You may know that I'm a big fan of the Test Data Builder pattern. So if your builder implements `IResponseBuilder<T>`, you can pass it directly to `RespondsWithJsonContent` or `RespondsWithODataResult` and Mockly will call `Build()` for you. The builders you already have for your domain objects can now also produce HTTP responses.

```csharp
mock.ForGet()
    .WithPath("/api/user")
    .RespondsWithJsonContent(new UserBuilder().WithId(123).WithName("John Doe"));
```

## Testing the unhappy paths

Testing the happy path is easy. The real bugs are usually in the code that deals with slow services, failures and retries. And that code is often not tested at all, because it's too much work to simulate those situations. With Mockly, it's usually one or two lines.

Consider the classic retry scenario where a service fails twice before it finally succeeds. You just chain the responses:

```csharp
mock.ForGet().WithPath("/api/resource")
    .RespondsWithStatus(HttpStatusCode.ServiceUnavailable)
    .ThenRespondsWithStatus(HttpStatusCode.ServiceUnavailable)
    .ThenRespondsWithStatus(HttpStatusCode.OK);
```

After the last response in the sequence, Mockly keeps returning that last one. And every `RespondsWith` method has a `ThenRespondsWith` variant, including the JSON, OData and delegate versions.

You can also limit how many times a mock will respond using `Once()`, `Twice()` or `Times(n)`. When a mock reaches its limit, Mockly skips it and tries the next one, in the order you created them. Notice that this is independent from the sequence. So a `Times(2)` mock with three responses will still stop matching after the second call.

To test timeouts or libraries like Polly, you can delay a response with `After(TimeSpan.FromSeconds(2))`. The delay respects the cancellation token, and if your `HttpClient.Timeout` is shorter than the delay, you get a `TaskCanceledException`, just like you would in production.

But not every failure has a status code. Connections get reset and requests time out. That's why you can use `ThrowsException<HttpRequestException>()`, `ThrowsException(someInstance)` or `TimesOut()` to simulate a failure at the network level. The exception goes directly to your code and is not converted into a 500 response. Mockly still records the request before throwing, so you can assert on it afterwards. And if you combine this with `Once()`, you have a first call that fails and a second one that succeeds.

### Authentication and rate limiting

Almost every API I integrate with requires some kind of authentication, and many of them will throttle you if you send too many requests. Testing that your code handles this correctly used to take quite a few lines of setup. Since version 1.13, Mockly has a couple of helpers for exactly these situations.

On the matching side, `WithBasicAuth("user", "pass")` checks for an `Authorization` header with the `Basic` scheme and those credentials. And `WithApiKey("X-Api-Key")` checks that the request carries an API key in that header. You can pass a wildcard pattern as a second argument if the value matters as well.

```csharp
mock.ForGet("/api/secure").WithBasicAuth("user", "pass");
mock.ForGet("/api/secure").WithApiKey("X-Api-Key", "secret-*");
```

On the response side, `RespondsWithOAuthToken` returns a proper OAuth2 token response with the `access_token`, `token_type` and `expires_in` fields. Real client libraries that expect a token endpoint will happily accept it. The token type defaults to `Bearer`.

```csharp
mock.ForPost("/token").RespondsWithOAuthToken("abc", expiresIn: TimeSpan.FromMinutes(5));
```

Then there is the classic token refresh test. Your code calls an API, gets a 401 because its token expired, refreshes the token and tries again. `RespondsWithUnauthorizedThenSuccess` returns a 401 for the first request and a 200 with the JSON you pass for every request after that.

```csharp
mock.ForGet("/api/data").RespondsWithUnauthorizedThenSuccess("""{"value":42}""");
```

And to test how your code deals with throttling, `RespondsWithRateLimit` returns a 429 (Too Many Requests) with a `Retry-After` header. Since it returns the same builder as the other responses, you can chain a successful response after it. This makes it easy to verify that resilience libraries like `Microsoft.Extensions.Http.Resilience` actually wait and retry the way you configured them.

```csharp
mock.ForGet("/api/data")
    .RespondsWithRateLimit(retryAfter: TimeSpan.FromSeconds(2))
    .ThenRespondsWithStatus(HttpStatusCode.OK);
```

## Failing fast and telling you why

By default, `FailOnUnexpectedCalls` is `true`, which means that any request that doesn't match a mock throws an `UnexpectedRequestException`. I did that on purpose. If a forgotten mock would silently return a 404, your test would probably fail much later with a message that has nothing to do with the real cause. You can turn it off if you really want that 404, but I would not recommend it.

Throwing early is only half of the story though. The exception must also explain what went wrong. And I think this is one of the most important parts of Mockly. When a request doesn't match, Mockly looks for the mock that came closest and shows you, criterion by criterion, what matched and what didn't:

```
Unexpected request to:
  POST https://api.example.com/api/users

Closest matching mock:
  POST https://*/api/users
    method       ✓ POST
    scheme/host  ✓ api.example.com
    path         ✓ /api/users
    query        ✓ (none)
    header       ✗ expected "X-Tenant: acme" but the request had no such header
    body         ✗ expected property "role" to be "Admin" but found "User"

Registered mocks:
 - POST https://*/api/users where header "X-Tenant" matches "acme"
```

You don't need a debugger to see that the tenant header is missing and the role is wrong.

Sometimes the problem is not a single request, but the entire conversation. Maybe a mock returned the wrong data, or the order of the calls was off. In that case, you can write `mock.GetTrafficReport()` to your test output. It lists every request the mock has handled, with the status code and whether it was expected or not.

```
Traffic report (3 request(s)):
  #1 GET https://api.example.com/api/users -> 200 OK
  #2 POST https://api.example.com/api/users -> 404 NotFound (unexpected)
      body: {"role":"Admin"}
  #3 GET https://api.example.com/api/users/123 -> 200 OK
```

## Assertions that explain your intent

Since I'm also the author of [Fluent Assertions](https://fluentassertions.com), it will not surprise you that Mockly has its own set of assertions. You can use them on the `HttpMock` itself, on a single mock, on a collection of captured requests and on a single request.

At the highest level, you can verify that all mocks were called, or that they were first called in a particular order:

```csharp
mock.Should().HaveAllRequestsCalled();
mock.Should().HaveCalledInOrder(mock1, mock2);
```

`HaveCalledInOrder` only looks at the first call of each mock. So it will still pass if a mock was called multiple times, as long as the first calls happened in the right order. And if you keep a reference to a single mock, you can use `HaveBeenCalled()`, `HaveBeenCalledTimes(1)` or `NotHaveBeenCalled()`. Like everything in Fluent Assertions, they accept a `because` to explain why you expect that.

### Inspecting what was sent

Mockly captures every request, including the headers, the body and a timestamp, in `mock.Requests`. But you can also collect the requests of one particular mock in a `RequestCollection` of your own:

```csharp
var patches = new RequestCollection();
mock.ForPatch()
    .WithPath("/api/update")
    .CollectingRequestsIn(patches)
    .RespondsWithStatus(HttpStatusCode.NoContent);
```

On such a collection you can use all the normal collection assertions, but also a couple that are specific for HTTP traffic:

```csharp
captured.Should().ContainRequestFor("POST /api/data");
captured.Should().NotContainRequestFor(HttpMethod.Delete, "/api/data");
captured.Should().ContainRequestsFor("/api/data", Exactly.Times(2));
captured.Should().ContainRequestsInOrder("/api/first", "/api/second");
captured.Should().AllHaveBeenSentWithin(TimeSpan.FromSeconds(1));
mock.Requests.Should().NotContainUnexpectedCalls();
```

After `ContainRequest()` or `ContainRequestFor()`, you can continue to assert on the headers, the bearer token, the query string, form fields, the body and even the headers of the response that Mockly sent back.

```csharp
captured.Should().ContainRequestFor("/api/data")
    .WithHeader("X-Custom-Header", "Expected*")
    .And.WithBody("*part-of-body*")
    .And.WithBearerToken("expected-token");
```

For JSON bodies, you can compare against a JSON string using `WithBodyMatchingJson`, or against an object graph using `WithBodyEquivalentTo`, which uses the same logic as `BeEquivalentTo`. You can also check for a subset of the top-level properties with `WithBodyHavingPropertiesOf`, for exactly those properties with `WithBodyHavingPropertiesEqualTo`, or for a single one with `WithBodyHavingProperty`.

Something I added with security in mind are the `Without` assertions. They verify that none of the matching requests contain certain data:

```csharp
captured.Should().ContainRequestFor("/api/users")
    .WithoutHeader("X-Internal-Token")
    .And.WithoutQueryParam("debug")
    .And.WithoutBodyProperty("password");
```

This is a simple way to prove that your code doesn't accidentally send an internal token or a password to some external service.

And finally, you can check an individual captured request with `BeExpected()`, `BeUnexpected()` or `BeASimulatedFailure()`. That last one is for requests that were handled by a mock configured with `ThrowsException` or `TimesOut`.

## Recording and replaying real traffic

Setting up mocks by hand is fine for a couple of endpoints. But if you depend on a large third-party API, it's often easier to start from real traffic. That's why Mockly can record and replay. When you call `RecordingTo`, all requests that don't match a mock are sent to the real service, and both the request and the response are written to a file.

```csharp
using var mock = new HttpMock()
    .RecordingTo("Recordings/github.json");
```

The file uses the [HTTP Archive (HAR)](https://w3c.github.io/web-performance/specs/HAR/Overview.html) format, so other tools can read it too. Mocks you configured still take precedence, so only the unmatched requests end up in the recording. By default, Mockly replaces the values of the `Authorization`, `Cookie`, `Proxy-Authorization` and `Set-Cookie` headers with `[REDACTED]`, so you don't accidentally commit credentials to your repository. If you really need those values, you have to explicitly call `KeepSensitiveRecordingValues()` first.

Later, you can replay that file using `LoadRecordings("Recordings/github.json")` without any network connection. Mockly finds the right recording by the HTTP method and the exact URL, and if the recording has a body, the body must match as well. Requests without a recording are treated as unexpected, unless you add `PassThroughUnmatched` to send them to the real service.

## Some smaller details

There are a few more things that I think are worth knowing about.

- By default, Mockly reads the entire request body into memory so that matchers and assertions can inspect it, both as text through `Body` and as raw bytes through `RawBody`. If you deal with very large or streaming content, you can set `PrefetchBody` to `false`. Just realize that the body assertions will then fail, because the body is no longer available.
- You can add more mocks after you created the client. The same client will pick them up.
- When you configure a new mock, Mockly reuses some of the settings of the previous one, like the scheme and the host. You can prevent that using `Reset()`. And `Clear()` removes all mocks.
- The regular expressions that Mockly generates from your wildcard patterns are cached, so a large test suite doesn't have to compile them over and over again.

## So is this the end of integration testing?

No, of course not. Mockly is not meant to replace tests against the real thing. It's meant for those tests where you consciously decided that the HTTP call is the boundary of what you're testing. For those tests, I want a setup that I can read in a couple of seconds. I want the unhappy paths to be as easy to simulate as the happy path. And when a test fails, I want the failure message to tell me what went wrong, without having to start a debugger. Everything in Mockly was built with those three goals in mind.

Mockly is free and open-source. You can find the full documentation at [mockly.org](https://mockly.org) and the source code on [GitHub](https://github.com/dennisdoomen/mockly). If you miss something, or you run into a situation that Mockly doesn't handle well, please open an issue. I'm very curious to hear how you use it.

## About me

I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 28 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections, and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastodon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).
