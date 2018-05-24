---
title: "Event Sourcing in .NET - Building fast autonomous projections"
header:
  teaser: "/assets/images/posts/2018-05-02_example.png"
tags:
- projections
- liquidprojections
- event sourcing
- cqrs
---

## The characteristics of a great projection implementation
Over the course of the last two years I've written [numerous](https://www.continuousimprover.com/tags/#event-sourcing) articles on the good, the bad and the ugly of Event Sourcing as well as on our experiences building and maintaining a distributed enterprise-class based on this increasingly popular architecture style. One [particular post](https://www.continuousimprover.com/2017/02/the-good-of-event-sourcing-projections.html) ended up being a kind of retrospective on how we used to build projections and what we've learned from that. The gist of it is that I believe projectors (or denormalizers if you wish) must have all the autonomy to decide on how it does its work and when. So whether or not a particular projector runs in memory, uses a document database or can benefit from a traditional OR/M is a decision that should only concern that particular projector. As a consequence, it becomes pretty evident that each projector should be able to run at its own pace and restart itself when the need arises. 

Another aspect of our profession that kept me busy last year is the notion of building libraries in a way that prevents you from ending up in a dependency hell. This was triggered by a [book](https://leanpub.com/principles-of-package-design) I read on the [Principles of Package Design](https://www.continuousimprover.com/2016/10/principles-for-successful-package.html) and which has fundamentally changed the way I look at software design. Inheritance for instance, is something I try to avoid, unless there's a real functional relationship between the parent and its inheritors. A lot of libraries tend to ease the adoption by introducing base-classes that should get you going pretty fast. However, quite often these tend to hide too much magic and force you in a certain direction. And if you need something that the library wasn't designed for, you're either stuck or you have to fork the library and create your own version. With that said, let me introduce [Liquid Projections for .NET](https://github.com/liquidprojections/LiquidProjections).

## Introducing Liquid Projections
Liquid Projections (or LP for short) is a set of highly efficient building blocks that each provide value on their own, but shine when used together to build synchronous and asynchronous projectors. It's the culmination of years of (painful) experiences and has been battle-tested in production for almost two years now. It's distributed as a collection of NuGet packages that ensure you only need to take dependencies on things you really need. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2018-05-18-overview.png" class="align-center" />

With the infinite knowledge of former colleague Ihar Bury (now Google), it fully embraces the `async` programming model and has been designed to run on as many .NET platforms as possible. As with all my open-source projects, it uses [Semantic Versioning](https://semver.org/) as well as Semantic Release Notes. We've been going through several breaking changes and many bug fixes, so we've finally reached calmer waters. Apart from some occasional talk on monoliths, Event Sourcing and [micro-services](https://www.slideshare.net/dennisdoomen/decomposing-the-monolith-into-embeddable-microservices-using-owin-webhooks-event-sourcing-and-the-onion-architecture), this is also the reason why I haven't been actively talking about this library yet. You may wonder where the name is coming from. But if you know my other open-source projects, [Fluent Assertions](https://fluentassertions.com/) and [Fluid Caching](https://github.com/dennisdoomen/FluidCaching), you may see the resemblence…. 

## A basic event map
Let's start with the first building block: the `EventMapBuilder<TContext>`. Its exposed by the `LiquidProjections` [NuGet](http://nuget.org/packages/liquidprojections) package and allows you to map events to lambda expressions without imposing any kind of structure to the way your projector is supposed to work. The `TContext` can be anything and is supposed to be passed to the map at run-time. It allows you to provide the mapping code with any kind of relevant information. In its simplest form, such a definition might like look like this:

```csharp
var mapBuilder = new EventMapBuilder<MyContext>();

mapBuilder.Map<WarrantAssignedEvent>().As(ctx =>
{
	// Do something, possibly involving the ctx variable
});

mapBuilder.Map<CountryRegisteredEvent>().As(ctx =>
{
	// Do something else, possibly involving the ctx variable
});
```

You can add as many maps as you want. In fact, the map even supports inheritance, so you can define a map for a more generic event like this:

```csharp
mapBuilder.Map<IEvent>().As(ctx =>
{
	// Handles any type of event
});
```

LP will ensure calling both the generic as well as the specific map. Conditional maps are also supported. Simply inject a `When` statement taking an expression like this:

```csharp
mapBuilder
	.Map<LicenseGrantedEvent>()
	.When((@event, ctx) => Task.FromResult(@event.Country == Countries.Netherlands))
	.As(ctx =>
	{
		// Only do something with that license when it's intended for a particular country 
	});
```

If you want to apply a similar filter on the entire map, use the `Where ` method on the map builder itself.

```csharp
var mapBuilder = new EventMapBuilder<ProjectionContext>()
	.Where(async (@event, ctx) =>
	{
		if (@event is IEvent knownEvent)
		{
			// Ignore any events from more than a year ago
			return knownEvent.OccurredAt > DateTime.Now.Subtract(TimeSpan.FromDays(365));
		}
		else
		{
			return false;
		}
	});       

```

But however you decide to set-up your map, ultimately you'll need to tell the builder to construct the actual map by calling `Build`. This method takes an object that has a single property of type `CustomHandler` to represent the generic handler (or _projector_) that should be called for each event. In most cases this shouldn't be more complicated than:

```csharp
IEventMap<ProjectionContext> map = mapBuilder.Build(new ProjectorMap<ProjectionContext>
{
	Custom = (context, projector) => projector(); 
});
```

The `projector` parameter hides the specific projection logic that was mapped as well as the actual event, so that's why this example is so trivial. To complete the entire example, let's use use the map.

```csharp
await map.Handle(new LicenseGrantedEvent
{
	Country = Guid.NewGuid(),
	InitialState = "Pending",
	Kind = "Apache 2.0",
	Number = "fluentassertions-1.0"
}, projectionContext);
```

That's it. There's nothing more to it. Now, as I said, you _can_ use the `EventMapBuilder` and forgot the rest of Liquid Projections. But it becomes a little more interesting if you pass this map builder into the `Projector` class, another building block. It provides two nice features: exception handling with a retry policy and child projectors. The first one means that it will wrap any exception in a `ProjectionException`, include all metadata about the current (batch of) event(s) and pass that to a retry policy in the form of the `ShouldRetry` delegate:

`public delegate Task<bool> ShouldRetry(ProjectionException exception, int attempts);`

This makes it very trivial to implement an exponential back-off strategy. The second feature, child-projectors, is great for building look-ups. These are little projectors that are invoked by the parent projector whenever it processes a batch of events. They will receive the same `TContext` and thus can potentially run under the same 'transaction', whatever that means for your particular scenario. And if an exception happens in a child-projector, all relevant information is captured and passed to its parent. This is how this projector could be created and used:

```csharp
var projector = new Projector(mapBuilder)
{
	ShouldRetry = async (exception, attempts) =>
	{
		await Task.Delay(attempts ^ 2);

		return (attempts < 3);
	}
};

projector.Handle(batchOfEvents);
```

The `Projector` class has some specific requirements though. For instance, the `batchOfEvents` parameter takes a collection of `Transaction` objects. This class, defined in the `LiquidProjections.Abstractions` NuGet package, represent a group of ordered events that happened in the same functional transactional boundary and apply to the same [stream](https://martinfowler.com/eaaDev/EventSourcing.html) (e.g. an [aggregate](https://martinfowler.com/bliki/DDD_Aggregate.html) in [DDD](https://domainlanguage.com/ddd/). In [NEventStore])(http://neventstore.org/) for instance, this maps to a `Commit`. Additionally, the `EventMapBuilder` that you pass into to projector must be using a `TContext` that inherits `ProjectionContext`. This context provides important metadata about the events that it needs to handle the events and provide some correlation when exceptions occur. It exposes the following properties:

* The `TransactionId` is a `string` that uniquely identifies the transaction within the entire event store.
* The `StreamId` is a `string` that uniquely identifies the object to which the events in this transaction apply to.
* The `TimeStampUtc` represents the point in time at which the transaction was persisted.
* The `Checkpoint` is a incrementing non-consecutive number that allows us to unambiguously determine the order of the transactions. They also serve as a tracking point to determine until what point a projector has processed the transactions.* Most event stores support associating metadata to the events and transactions. These are captured by `EventHeaders` and `TransactionHeaders`. 

Again, as with everything in Liquid Projections, you don't have to use the `Projector`. It's perfectly fine to use the `EventMapBuilder` without the projector. 

## Connecting your projectors to an event store
I have not discussed the source of those events (and the transactions that wrap them yet), but `LiquidProjection.Abstractions` does define an abstraction for an event store. It looks like this:

```csharp
public delegate IDisposable CreateSubscription(long? lastProcessedCheckpoint, Subscriber subscriber, string subscriptionId);
```

So we basically require an event store to allow adding a subscription that starts at the provided `lastProcessedCheckpoint`, is identified by `subscriptionId` and notifies the subscriber through the properties of the `Subscriber` class:

```csharp
public class Subscriber
{
    public Func<IReadOnlyList<Transaction>, SubscriptionInfo, Task> HandleTransactions { get; set; }

    public Func<SubscriptionInfo, Task> NoSuchCheckpoint { get; set; }
}
```

Based on this definition you can tell that a subscriber is interested in two things. Which transactions it should process and whether or not it requested a checkpoint that (no longer) exists. This last part can be used by subscribers to detect that the event store  was rolled back to an earlier state (e.g. a restore of a backup) and trigger a rebuild of the projections. 

The [LiquidProjections.NEventStore](https://www.nuget.org/packages/LiquidProjections.NEventStore/) package provides an adapter for NEventStore that supports this contract. Since this is a passive event store that you have to poll regularly, it uses the [LiquidProjections.PollingEventStore](https://www.nuget.org/packages/LiquidProjections.PollingEventStore.Sources/) source-code package to implement a very efficient highly scalable adapter that supports numerous subscribers with ease. It uses an LRU cache to deal with subscribers running at a different pace without hitting the underlying database too often. And it even prefetches pages to make sure the events are already there by the time the projector processes a batch of events. 

There's also an in-memory implementation. This `MemoryEventSource` is provided by the [LiquidProjections.Testing](https://www.nuget.org/packages/LiquidProjections.Testing/) package and supports writing events in a synchronous fashion as well as asynchronously queueing up events for processing by one or more subscribers. It's heavily used by the unit tests that are used in the various Liquid Projections' repos.

## To dispatch or not to dispatch
You _can_ manually connect a projector to an implementation of the `CreateSubscription` delegate. But Liquid Projections offers another little building block that help you with that; the `Dispatcher`. It's constructor takes the `CreateSubscription` delegate as input and has a method named `Subscribe` that can be used by a projector to... well ...subscribe itself. Now why would you use this extra building block if you can do all of this directly? To understand the value of the `Dispatcher`, let's first set-up the dispatcher itself, something you usually do in your bootstrapping code. 

```csharp
public void Initialize(CreateSubscription subscribeToEventStore)
{
    var dispatcher = new Dispatcher(subscribeToEventStore);
    
    dispatcher.HandleException = async (exception, attempts, info) =>
    {
        // Log or track the exception somewhere
    
        if ((IsTransient(exception) && attempts < 3))
        {
            return ExceptionResolution.Retry;
        }
        else
        {
            return ExceptionResolution.Abort;
        }
    }
    
    dispatcher.SuccessHandler = async info => 
    {
        // Clear any error state for the projector identified by info.Id
    };
    
    return dispatcher;
}
```

As you can see, the `Dispatcher` is a useful building block that allows you to handle exceptions centrally, irrespective of the number of subscribers. But it has another nice feature that can be best illustrated with the following example projector.  

```csharp
public async Task Start(Dispatcher dispatcher)
{
    long? initialCheckpoint = await DetermineLastCheckpoint();
    
    var options = new SubscriptionOptions
    {
        Id = "Myprojector",
        RestartWhenAhead = true
        BeforeRestarting = async () => { await ClearAllProjections(); }
    }
    
    IDisposable subscription = dispatcher
        .Subscribe(initialCheckpoint, async (transactions, info) =>
        {
            await innerProjector.Handle(transactions, info.CancellationToken);
            
            await StoreLastCheckpoint(transactions.Last().Checkpoint)
            
        }, options);
}
```

Not only does this snippet show you the typical structure of a projector based on Liquid Projections, but it also highlights a neat little feature of the `Dispatcher` class: the ability to detect a projector that is ahead of the event store. So when the projector requests a subscription that starts at the `Transaction` with a particular particular checkpoint, the `Dispatcher` will pass on that request to the underlying event store. If the first transaction that the event store received has a lower checkpoint number than the one requested, it assumes the event store got rolled back to an earlier point (or restored from a backup). It will then give the subscriber a chance to handle that, e.g. by cleaning the entire projection table, before starting the subscription at checkpoint 0. 

As I said in the beginning of this post, my goal was to provide building blocks that don't force you in any direction. That's why some of the steps to connect the dots may feel a bit complicated. But understanding how those dots connect and how to take the bits and pieces you care about, is the key to get the most out of this library.

## Mapping creates, updates and deletes
Now that you understand how the building blocks of Liquid Projections work together, it's time to discuss an `EventMapBuilder` that is a more natural fit for _create_, _update_ and _delete_ operations. It's built on top of the `EventMapBuilder<TContext>` we started this post with, but adds the generic `TProjection` and `TKey` parameters. Let's see how this can be used.

```csharp
var mapBuilder = new EventMapBuilder<MyContext, DocumentProjection, string>();

mapBuilder
    .Map<WarrantAssignedEvent>()
    .AsCreateOf(anEvent => anEvent.Number)
    .Using((document, anEvent) =>
    {
        document.Type = "Warrant";
        document.Kind = anEvent.Kind;
        document.Country = anEvent.Country;
        document.State = anEvent.InitialState;
    });

mapBuilder
    .Map<StateTransitionedEvent>()
    .When(anEvent => anEvent.State != "Closed")
    .AsUpdateOf(anEvent => anEvent.DocumentNumber)
    .Using((document, anEvent) => document.State = anEvent.State);

mapBuilder
    .Map<DocumentArchivedEvent>()
    .AsDeleteOf(anEvent => anEvent.DocumentNumber);
```    

As you can see, this type of builder supports fluent methods like `AsCreateOf`, `AsUpdateOf`, `AsDeleteOf` and the original `As` from the inner `EventMapBuilder<TContext>`. Each of these methods has specialized capabilities such as `IgnoringDuplicates`, `ThrowingIfMissing` and much more to fine-tune the behavior of the map. And in addition to the `mapBuilder`-level `Where` filter, the individual mappings can be made conditional using the `When` method. 

Just like his simpler cousin, this builder has a `Build` method that takes a container object representing the actions to execute for the various types of maps. 

```csharp
var map = mapBuilder.Build(new ProjectorMap
{ 
    Create = async (key, context, projector, shouldOverride) => { ... }
    Update = async (key, context, projector, createIfMissing) => { ... }
    Delete = (key, context) => { ... } 
    Custom = (context, projector) => { ... } 
});
```
    
These work very well with the [NHibernate](https://github.com/liquidprojections/LiquidProjections.NHibernate/) and [RavenDB](https://github.com/liquidprojections/LiquidProjections.ravendb) building blocks which I'll talk about in a next post.

## Collecting statistics and predicting progress
Since most real-world applications of Liquid Projections will result in autonomous asynchronous projectors (at least, that's how we build them), at some point you may want to get some insights in how your projector is doing. What settings did it use (if any), how fast is it running and how long will it take to reach a certain checkpoint, and what significant issues have been logged. This is where another little building block comes into the picture, the `ProjectionStats`. It's a thread-safe class that you should set-up somewhere centrally and then use it in your projector.

```csharp
// Somewhere in your bootstrapping code.
var stats = new ProjectionStats(() => DateTime.UtcNow);

// Later, in your projector, track some arbitrary setting value under its key
stats.StoreProperty("CountByDocument", "some setting key", "some value");

// Track an important occurrence that happened within the scope of your projector
stats.LogEvent("CountByDocument", "some significant thing that happened");

// Track the checkpoint that was last processed by a projector
stats.TrackProgress("CountByDocument", currentCheckpoint);
``` 

That last method is pretty neat since it allows `ProjectionStats` to calculate the weighted average speed as well as the time to reach a particular checkpoint:

```csharp
float? transactionsPerSecond = stats.GetSpeed("CountByDocument");
TimeSpan? eta = stats.GetTimeToReach("CountByDocument", targetCheckpoint);
```

The speed is calculated by combining the speed of the projector over the last 10 minutes, but putting more weight on the speed in the last minute. This should give you a nice average that does not fluctuate too much without ignoring the progress from the last minute. If you care, you can check out the implementation details of the [algorithm in code](https://github.com/liquidprojections/LiquidProjections/blob/master/Src/LiquidProjections/Statistics/WeightedProjectionSpeedCalculator.cs). 

If you include the `LiquidProjections.Owin` package, you can even expose those statistics on your [OWIN](http://owin.org/) pipeline like this:

```csharp
public void Configure(IAppBuilder builder)
{
    ProjectionStats stats = // get singleton from somewhere
    
    builder.UseStatistics(stats);
}    
```

After this, sending a `GET` request like `http://localhost/projectionStats/CountByDocument` will give you something like:

```json
{
    "projectorId": "CountByDocument",
    "lastCheckpoint": 1000,
    "lastCheckpointUpdatedUtc": "2018-05-10T10:39:00Z",
    "properties": [{
        "key": "some setting key",
        "value": "some value",
        "lastUpdatedUtc": "2018-05-10T10:39:00Z"
    }],
    "eventsUrl": "http://localhost/projectionStats/CountByDocument/events"
}
```

Requesting the events for that projector using the URL stated by the `eventsUrl` property might give you:

```json
{
    "projectorId": "CountByDocument",
    "events": [{
        "body": "some significant thing that happened",
        "timestampUtc": "2017-07-10T10:39:00Z"
    }],
}
```

## So now what?
Pfew, this was a big one. I hope the underlying principles make sense to you all. In a next post, I'll talk about the specific packages for RavenDB and NHibernate, and why I still think an OR/M can help you build fast projectors. In terms of feature set I don't have any major ideas left. I am seeing a couple of patterns emerge from our production code, so I may decide to include those in the project in some form. But if I do, they will be _based_ on the existing building blocks. 

So what do you think? Do you agree with the principles on which Liquid Projections has been built? Would you consider using this library? If not, why not? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.