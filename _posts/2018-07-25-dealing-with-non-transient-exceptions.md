---
title: "Event Sourcing in .NET - Dealing with projection exceptions"
header:
  teaser: "/assets/images/posts/.png"
tags:
- projections
- liquidprojections
- event sourcing
- cqrs
- nhibernate
---

## Transient vs non-transient exceptions
If I have to name the single biggest flaw in adopting Event Sourcing, it must be our decision to rely on the synchronous dispatching pipeline of [NEventStore](https://github.com/NEventStore/NEventStore). It is based on the idea that every event will be processed by all projectors in a synchronous manner. As we use a RDBMS as our backing store, it'll use a `Dispatched` column to track that all projectors have succesfully processed the event(s) in a functional transaction. As a consequence of that, the exception handling strategy ended up being very simple and naive. When anything fails, all projection attempts fail and the involved event(s) is ignored. The projection work continues with the next event(s) in the stream. Then, after restarting the system, it'll first try to reproject those events. This can result in some older events being processed by a projector even though it already processed newer events. 

This approach directly opposes my current believes that projectors should be completely autonomous. They should be able to control the speed at which they project the events, what kind of storage they use (e.g. RDBMS, NoSQL database or some in-memory respresentation), but equally important, how to deal with exceptions. A great projector acknowledges the differences between transient exceptions - those that usually disappear after a while - and non-transient exceptions such as foreign key contraint violations, and act accordingly. For instance, a temporary database timeout or outage can often be handled by retrying with an exponential delay, before giving up.

## Dealing with corrupted projections
As I said in [my last post]({{ site.baseurl }}{% post_url 2018-07-15-using-orm-as-projection-library %}), _if_ you decide to use a RDBMS as your projector's storage, I believe something like NHIbernate still has a lot of value for projectors where the unit of work gives you a performance improvement. The NH projector facilitates that by creating an `ISession` per batch of `Transaction` instances. As a consequence of that, this batch will most probably affect a lot of the underlying projections. NH will not flush the changes made to the session until it sees a need for that. 

For instance, when a query is ran against a table and NH has some unflushed changes in the session, it'll first flush those changes back to the database. If not then, then those changes are flushed to the database when the database transaction is committed or when your code calls `ISession.Flush` explicitly. So all in all, database contraints can kick in at any time, making it quite difficult to figure out what particular event was the culprit of the exception that represents it. Obviously, LP will try to collect as much as possible information before it wraps that exception into a [ProjectionException](https://github.com/liquidprojections/LiquidProjections/blob/master/Src/LiquidProjections/ProjectionException.cs), but that might not be enough. 

To help you with that, the NH version of LP has one extra exception handling mode called `RetryIndividual`. When your handler returns this as the `ExceptionResolution`, the `NHibernateProjector` will process each transaction in a separate database transaction, one by one. By doing this, it'll be much easier to figure out which specific `Transaction` caused the database exception. In fact, if there's a clear relation between the `StreamId` of that transaction and the underlying projection, you can use that to mark that projection as corrupt. To illustrate this, consider the following snippet. 

```csharp
private async Task<ExceptionResolution> OnException(ProjectionException exception, int attempts, CancellationToken cancellationToken)
{
    if (IsTransient(exception))
    {
        // If this is one of the first three attempts, just retry the 
        // entire batch of transactions. Maybe the exception resolves by itself. Otherwise just abort.
        if (attempts < 3)
        {
            await Task.Delay(TimeSpan.FromSeconds(attempts ^ 2));

            return ExceptionResolution.Retry;
        }
        else
        {
            return ExceptionResolution.Abort;
        }
    }
    else
    {
        if (exception.TransactionBatch.Count > 1)
        {
            // If we have more than one transition, let's try to run them 
            // one by one to trace down the one that really fails.
            return ExceptionResolution.RetryIndividual;
        }
        else
        {
            // So we found the failing transaction. So let's mark it the affected 
            // projection as corrupt and skip the transaction.
            using (var session = sessionFactory())
            {
                string failingStreamId =  exception.TransactionBatch.Single().StreamId;
                if (failingStreamId != null)
                {
                    var projection = session.Query<DocumentCountProjection>().SingleOrDefault(x => x.Id == failingStreamId);
                    if (projection != null)
                    {
                        projection.Corrupt = true;
                        cache.Clear();
                        session.Flush();
                    }
                }
            }

            return ExceptionResolution.Ignore;
        }
    }
}
```

If the exception is transient (e.g. a timeout or temporary database unavailability), the exception handler will retry up to three times with an increasing delay. But if the exception is non-transient, like a unique key violation or a foreign key constraint, it'll use the aforementioned option to trace down the transaction that caused the violation. By mapping the stream ID of that transaction to the projection, it'll mark the projection as corrupt, using some arbitrary property I added to the projection, and ignore the exception. If there's no direct correlation between your events and the projection, you're out of luck obviously. But for us this worked quite well.

This technique allows the projector to continue processing with the caveat that that specific projection is now corrupt. But it's very likely that any successive events related to that same projection can no longer be processed in a reliable way. That's why the `NHibernateProjector` has a `Filter` property to ignore any events associated with corrupted projections:

```csharp
projector = new NHibernateProjector<DocumentCountProjection, string, ProjectorState>(
    sessionFactory, MapBuilder,
    (p, id) => p.Id = id)
{
    // Make sure that we no longer process events associated with corrupted projections. 
    Filter = p => !p.Corrupt
};
```

## Summary
So how do you deal with transient and non-transient exceptions that happen during your projection work? And are you still using an RDBMS as your storage technique or did you adopt a NoSQL solution by now? Let me know by commenting below.  Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.