---
title: "Event Sourcing in .NET - Using an OR/M as a projection library"
header:
  teaser: "/assets/images/posts/2018-07-15-nhibernate.png"
tags:
- projections
- liquidprojections
- event sourcing
- nhibernate
- object-relational mapper
---

## I thought we didn't need OR/Ms anymore? 
A common advantage of adopting Event Sourcing is that it solves the impedance mismatch between object oriented code and the relation database model. And because of that, Object/Relational Mappers (OR/Ms) have become obsolete. While I agree with the first statement, I actually think an OR/M like NHibernate can have a lot of value under certain circumstances. Sure, [properly designed projections](https://www.continuousimprover.com/2016/06/event-sourcing-from-trenches-projections.html) will be optimized in such a way that querying the data results in a data set which schema matches the data on the screen or the data exposed by an API as close as possible. But building such a projection usually involves less-than-trivial writes that may or may not require lookup tables used for joining or collecting data from different entities. 

Now, imagine a projector that processes a lot of events in a batch, and those events tend to affect the same projection record multiple times. A raw SQL-based projector would typically issue multiple `UPDATE` statements on the same record. Those are a lot of round-trips to the database. And what if it's rebuilding a projection that ends up being deleted again? It is possible that a whole range of CREATE and UPDATE statements are followed by a DELETE statement. So even though we don't have the object-relation impedance anymore, and you may not need to support multiple database vendors, there's one more nice feature: the [unit of work](https://martinfowler.com/eaaCatalog/unitOfWork.html). 

## Ok. That makes kind of sense. Now what about NHibernate?
NHibernate's implementation of this pattern is represented by the `ISession` interface and serves as a [first-level cache](http://nhibernate.info/previous-doc/v5.0/single/index.html#performance-sessioncache). As long as you keep this session open, most of the CRUD operations will be handled in memory until you _flush_ that session back to the database, thereby saving tens and tens of unnecessary queries. If you _do_ query on some projection that has uncommitted changes (it is as we say, _dirty_), NHibernate will automatically flush your changes back to the database before running the query. And if that's too slow for you, you can always use [DML operations](https://ayende.com/blog/4037/nhibernate-executable-dml) to bypass the session and run complicated UPDATE or DELETE statements directly against the database. Combine that with advanced features like dynamic updates where only the changed columns are included in the updates and you have a pretty powerful caching technique. And did I mention second-level caching (or cross-session caching)? Caching is hard, so having something battle-tested that you can plug in using any of the available caching providers can speed up the projector even more. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2018-07-15-nhibernate.png" class="align-center" style="width:300px"/>

But NHibernate has one more neat feature; the ability to generate the database schema from the class maps. Since projectors are supposed to be autonomous _things_ that can decide when to rebuild their persistent store, being able to define your mapping in code and then have NHibernate generate your schema on-the-fly is pretty powerful. We do that in some of our projects where we keep a separate file-backed Sqlite database per projector. As soon as the projector creates a connection to that database, Nhibernate will apply the initial schema from the class maps in the auto-created Sqlite database file. 

## Great. Now how do I build projectors with it?
Knowing all this, it should not come as a surprise that one of the extension packages to LiquidProjections, `LiquidProjections.NHibernate` is there to give you the `NHibernateProjector` a powerful building block with a lot of out-of-the-box features. The class has the following definition:

```csharp
class NHibernateProjector<TProjection, TKey, TState>
    where TProjection : class, new()
    where TState : class, IProjectorState, new()
```

Its constructor has the following signature:

```csharp
NHibernateProjector(
    Func<ISession> sessionFactory,
    IEventMapBuilder<TProjection, TKey, NHibernateProjectionContext> mapBuilder, Action<TProjection, TKey> setIdentity,
    IEnumerable<INHibernateChildProjector> children = null)
```

As you can see, the projector needs a couple of ingredients. First, it needs to know the type of the class to project (`TProjection`). It doesn't have any specific requirements for it other than that it should be possible to request an instance through NH by the key of type `TKey` _and_ it needs to be able (`setIdentity`) to give a new projection an identity. 

Since the projector must be able to create and destroy `ISession`s when it needs to, it expects to receive a factory to do exactly that. How you configure that session and whether or not you reuse it is up to your _actual_ projector (remember *composition over inheritance*?). 

Obviously this projector can't do much without being told how to handle certain events. That's why it takes the `IEventMapBuiler` that I discussed extensively in the [introductionary blog post](https://www.continuousimprover.com/2018/05/building-projections-in-dotnet.html). The only special thing here is that it expects a `NHibernateProjectionContext` or something that derives from it. This class extends the `ProjectionContext` with an extra property exposing the `ISession` to the event handlers. 

Just like the `Projector` building block, this one is quite passive. In other words, it's the encompassing projector that needs to subscribe to some event store as of a certain checkpoint number and pass the resulting transactions to the `Handle` method. So that might imply that it also the job of that same projector to update the checkpoint after those transactions have been processed. But there are two little details to consider in this: batches and transactions. 

By default, the `NHibernateProjector` will create a new session and a new database transaction for every new `Transaction` that you give it. But by increasing the `BatchSize` you can tune the projector to benefit more from the unit-of-work. But that also means that a big collection of transactions will be broken down into smaller batches. Now imagine that the first few batches successfully commit to the database, but a later one causes some kind of _foreign key_ or _unique key constraint_ exception. If you would normally update the checkpoint _after_ the NH projector has partially processed your transactions, the exception may cause this step to be skipped. Since one or more batches were already committed to the database you end up with an inconsistent state. 

This is why the NH projector itself will need to update the checkpoint after each batch of transactions _within_ a database transaction. This is where the `TState` type parameter comes into play. It represents an NH-mapped class that implements `IProjectorState` and which is used by the projector to store the last processed `Checkpoint` along with a `LastUpdateUtc` under the `Id` that identifies your projector. By default this is the name of the class, but you can change that using the `StateKey` property. 

As an example, this is what the initialization code of a projector based on the `NHibernateProjector` can look like:

```csharp
public void Initialize()
{
    documentProjector = new NHibernateProjector<DocumentCountProjection, string, ProjectorState>(
        sessionFactory, mapBuilder,
        (projection, identity) => projection.Id = identity)
    {
        BatchSize = 20,

        // Make sure that we no longer process corrupted projections. 
        Filter = p => !p.Corrupt
    };
}

public void Start()
{
    long? lastCheckpoint = documentProjector.GetLastCheckpoint();

    dispatcher.Subscribe(lastCheckpoint, async (transactions, info) =>
    {
        await documentProjector.Handle(transactions, info.CancellationToken ?? CancellationToken.None);
    });
}
```

## So what about exception handling?
Glad you asked. Considering the auto-flushing behavior of an NH session, the batching and the way this affects the transaction boundaries, you can imagine that exception handling can become quite interesting. Unlike a raw SQL statement, when using sessions, exceptions can happen while processing a single event, during a query, or while flushing the changes in the session back to the database. Correlating a database-level unique key constraint violation to a particular event or `Transaction` isn't a trivial thing.

To handle exceptions, you set-up the `ExceptionHandler` property of the NH projector with a method that has the following signature:

```csharp
delegate Task<ExceptionResolution> HandleException(
    ProjectionException exception, 
    int attempts, 
    CancellationToken cancellationToken)
```

This method allows you to decide how to deal with a particular exception. For instance, if the exception type denotes some kind of recoverable error, you can decide to return `ExceptionResolution.Retry` for the first few `attempts`, and then abort the attempts by returning `ExceptionResolution.Abort`. Since the signature is `async` all the way, you can even implement a simple [exponential back-off algorithm](https://dzone.com/articles/understanding-retry-pattern-with-exponential-back) by adding something like this:

```csharp
await Task.Delay(TimeSpan.FromSeconds(attempts ^ 2));
```

## Any other neat stuff?
Similar to the `Projector` class in the main project, the `NHibernateProjector` supports child-projectors to maintain one or more look-ups or related projections. The batches of transactions that the parent projector processes will also get passed to the child projector as part of the same `ISession`. This ensures that the child projections will stay consistent with the parent. 

Something we do often in our own projects is to track additional information in addition to the transaction checkpoint and last timestamp defined by the `IProjectorState` interface. For instance, we track whether a projector is rebuilding itself after a schema change and use the `ProjectorStats` class from the main project to track the projection speed. However, to ensure this information is consistent with the rest of the projector state, we would like to update this as part of the same `ISession` that is projecting the events. This is where the `EnrichState` property of the `NHibernateProjector` comes into play. It takes a delegate with the following signature:

```csharp
public delegate void EnrichState<in TState>(TState state, Transaction transaction) where TState : IProjectorState;
```

Which could be used like this:

```csharp
projector.EnrichState = (state, transaction) =>
{
    state.LastStreamId = transaction.StreamId;
};
```

This is also the main reason why the `NHibernateProjector` takes a generic `TState` parameter. As long as it implements `IProjectorState`, you can add whatever additional information you need and use the `EnrichtState` hook to update the projector state at the right point of time. 

The projector also provides a simple caching mechanism in the form of the `IProjectionCache` and ships with the `LruProjectionCache` based on the [FluidCaching](https://www.nuget.org/packages/FluidCaching.Sources/) project. The `IProjectionCache` is meant for simple scenarios and thus has some limitations you need to be aware of.
   * If the projector performs database modifications directly on the NHibernate `ISession`, that projector must make sure the cache is updated or cleared accordingly.
   * The cache doesn't understand relationships where a projection refers to another projection maintained by the same projector. For instance, a projector that maintains a graph of parents and children, where a child is also a (direct or indirect) parent must use a more advanced type of caching. 
This kind of caching provides a lot of control over what your specific projector caches or not. However, if you need more advanced caching, for instance, to work around the above limitations, check out NHibernate's [Second Level Caching](http://nhibernate.info/doc/nhibernate-reference/caches.html) feature.

## That's it folks
In a next post, I'll talk a bit on how to handle exceptions by marking projections as corrupt. But in the mean time, I'd love to hear your thoughts on using an OR/M to efficiently (re)build projections. Would you follow the same approach? Or do you think projections should use raw SQL only? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
