---

title: The curious case of a deadlock in Autofac
date: '2015-01-14T19:42:00.001+01:00'

tags:
- autofac
- threading
modified_time: '2015-01-14T19:58:22.123+01:00'
---

It has been more than [two years]({{ site.baseurl }}{% link _posts/2011/2011-09-28-silverlight-cookbook-switching-to.html %}) since we switched from Microsoft Unity to [Autofac](https://autofac.org/) and we haven't regretted this a single day. Not only is the resolution performance much better than Unity, but it’s the feature set, in particular the relation types, that makes a world of difference. In terms of concurrency, there's not much to think about it. Autofac's concurrency model has been designed to be thread-safe from the ground up. In fact, there's only a single scenario in where you have to be careful what you do. And that's exactly where we went wrong… 

The case this post is about involves the `ContainerBuilder`'s fancy factory registration API. Consider the following statement. 

```csharp
var builder = new ContainerBuilder(); 
builder.RegisterIQueryProcessor(ctx => new QueryProcessor()).SingleInstance();
```

This tells Autofac that every time my code needs an instance of `IQueryProcessor` (either through a constructor or property dependency or through an explicit call to `Resolve`), it should return the same instance of `QueryProcessor` initialized with a particular storage option. So far so good, but imagine that my `QueryProcessor` class has a dependency on the imaginary `ICachingStrategy` abstraction that has been registered upfront. I can change my registration to this.

```csharp
builder.RegisterIQueryProcessor(ctx => 
  new QueryProcessor(ctx.ResolveICachingStrategy())).SingleInstance
``` 
  
The `ctx` variable gives me access to the container so that I can resolve any other dependencies from it. If you carefully read the [concurrency section](https://autofac.readthedocs.org/en/latest/advanced/concurrency.html) of Autofac's wiki, you'll notice that this `ctx` variable represents a temporary copy of the actual container. In other words, this container is only available during the invocation of the lambda expression. And that's exactly how Autofac manages to be so thread-safe. Any locking that would normally happen in the global container now happens in a local temporary container and will not affect any other resolutions, nor prevent reentrance. 

Considering this temporary nature, the following might be very dangerous.

```csharp
builder.RegisterIQueryProcessor(ctx => new QueryProcessor(ctx
  .SingleInstance(); 
```

Notice that I'm passing the `ctx` variable into the constructor. Now, it highly depends on what the constructor is doing here, but assume for now that it is storing that container reference in a private field and using that to resolve other dependencies at a much later stage. Didn't I just mention that we're dealing with a temporary container? Indeed. It's very likely that that resolution attempt will blow up in your face! So how do you fix this? Like this:

```csharp
builder.RegisterIQueryProcessor(ctx => 
  new QueryProcessor(ctx.ResolveIComponentContext()).SingleInstance()  
```

Resolving the `IComponentContext` (Autofac's abstraction for a container) returns the actual [global](https://nblumhardt.com/2011/01/an-autofac-lifetime-primer/) container that you can safely use at a later point of time. So to summarize, use the temporary container for resolving dependencies needed at construction time, but use the global container to resolve any run-time dependencies. Guess what we did wrong…. 

So what are your experiences with Autofac? Share them here or let me know by tweeting me at [@ddoomen](https://twitter.com/).