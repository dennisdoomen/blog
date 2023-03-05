---

title: "Using docker to write automated tests against a real database"
excerpt: "It's 2023, so what if you could run SQL Server in a Linux docker container just before your tests start?"

tags:
- docker
- unit-testing
- automated-testing

---

Whether you work with SQL Server, PostgreSQL or some other database that can't run in-memory, you inevitably end up having a need to write automated tests that cover the specific queries and database operations your product needs. Whether or not you call those "unit" or "integration tests" is irrelevant to me, but I often see developers introduce abstractions like a generic `IPermitRepository`, a `IDatabaseManager` (with `IDatabaseManagerFactory`), or worse, their own expression-based abstraction on top of LINQ. Introducing abstractions is not a bad thing by itself and can help create natural seams in your architecture. But I've seen plenty of examples where people overengineer their code "because of SOLID". 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/docker.png" class="align-center"/> 

That being said, in the ideal world, you want to make sure your tests also cover the database. Entity Framework supports [both a simplified in-memory provider as well as a built-in SQLite provider](https://learn.microsoft.com/en-us/ef/core/testing/) to help you with that. And using SQLite is not a bad idea at all. In a previous life, where we were using the excellent [NHibernate](https://nhibernate.info/), we did the same thing. But again, it's not the real deal, and it's 2023, so what if you could run SQL Server in a Linux docker container just before your tests start? 

With the open-source library [Testcontainers for .NET](https://dotnet.testcontainers.org/), this becomes rather trivial. Consider the below code snippet. 

```csharp
_sqlServerContainer = new ContainerBuilder()
    .WithImage("mcr.microsoft.com/mssql/server:2019-GA-ubuntu-16.04")
    .WithPortBinding(1443, assignRandomHostPort: true)
    .WithEnvironment("ACCEPT_EULA", "Y")
    .WithEnvironment("SA_PASSWORD", Password)
    .WithCleanUp(cleanUp: true)
    .WithWaitStrategy(Wait.ForUnixContainer()
        .UntilOperationIsSucceeded(() => HealthCheck(CancellationToken.None).GetAwaiter().GetResult(),
            10))
    .Build();

    await _sqlServerContainer.StartAsync();
```

This will build and start a new Ubuntu Linux container running SQL Server 2019 on the first available mapped to internal port 1433. Then it will wait until the custom `HealthCheck` operation succeeds, but not more than 10 times. In our case, we use the `HealthCheck` method to try to connect to the container-hosted SQL Server instance. And how do we get the port to connect to? Simple, using `_sqlServerContainer.GetMappedPublicPort(1433)`.

And what about cleaning up after ourselves? Well, another nice feature is that the `WithCleanUp` will launch a second container which only task is to monitor the first one for inactivity. If the SQL Server container has been inactive for a certain amount of time, all containers will be shut down automatically. 

To use this in your tests, we've created a nice wrapper around all of this called `DockerMsSqlServerDatabase`. It'll create and start the container, create an empty database, expose the connection string, and delete the database during the dispose. If another test reuses the container before the timeout expires, it'll reuse it. Otherwise a new one will be created on another free port. You can find the full implementation [here](https://gist.github.com/dennisdoomen/9a97e07a4c4a8f2eef3af5ac293d6759). With that, you can use the wrapper in your tests like this:

```csharp
public class DiagramIndexServiceSpecs : IAsyncLifetime
{
    private DockerMsSqlServerDatabase _databaseServer;

    public async Task InitializeAsync()
    {
        _databaseServer = await DockerMsSqlServerDatabase.Create();
    }

    public async Task DisposeAsync()
    {
        await _databaseServer.DisposeAsync();
    }
}
```

So how do you run automated tests that need a real database? Do you think my proposal is useful? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
