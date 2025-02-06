---

title: "Static and dynamic conditions in C#/Nuke build pipelines"
excerpt: "How you can use Nuke's OnlyWhenStatic and OnlyWhenDynamic to handle the more complicated build pipelines in C#"
tags:
---

Sometimes you run into a little open-source project that makes you wonder how you ever lived without it. [Nuke](https://nuke.build/), the C#-based build pipeline framework, is definitely one of them. If you haven't seen Nuke yet, check out [my article]({% post_url /2020/2020-03-01-reasons-for-adopting-nuke %}) for a in-depth explanation. For now, consider the following excerpt.


```csharp    
class Build : NukeBuild
{
    public static int Main() => Execute<Build>(x => x.TargetA);

    [Parameter] string AzureToken

    Target Prepare => _ => _
        .Requires(() => AzureToken)
        .Executes(() =>
        {
            // Do something useful with SomeParameter
        });

    Target Deploy => _ => _
        .DependsOn(Prepare)
        .Executes(() =>
        {
            // Do something else
        });
```

This is a simple example in which target `Deploy` has a dependency on target `Prepare`. When you tell Nuke to execute `Deploy` through `build.ps1` (or `nuke` if you've installed it as [global tool](https://nuke.build/docs/getting-started/installation/)), it will make sure that `Prepare` is executed before `Deploy`. And since `Prepare` has a required parameter, `SomeParameter`, it'll will make sure it has a non-`null` value. If not, it will either throw, or if you're running the script from a command prompt, prompt you to enter a value. 

Now, imagine that you want to execute a target only when a certain condition is met. For example, when you have another target that uses runtime logic to set a field to a certain value that another target should depend on:

```csharp
    bool HasChanges;

    Target DetermineChanges => _ => _
        .Executes(() =>
        {
            HasChanges = // execute some run-time logic to set this field
        });
```

There's actually two ways to make a target conditionally dependent on the value of `HasChanges`. One is called `OnlyWhenStatic` and the other `OnlyWhenDynamic`. If you've been building non-trivial build scripts using Nuke, at some point you must have wondered what the subtle difference is between those two. To understand that, look at the below version of `Deploy`:

```csharp
    Target Deploy => _ => _
        .DependsOn(Prepare)
        .DependsOn(DetermineChanges)
        .OnlyWhenDynamic(() => HasChanges)
        .Executes(() =>
        {
        });
```

Since Nuke will build a dependency graph of all the targets before executing any of them, in this case, it will make sure that both the dependencies `Deploy` and `DetermineChanges` have executed before it even considers to run `Deploy`. And that's nice since it gives `DetermineChanges` a chance to run that logic I mentioned and set `HasChanges` to an appropriate value. Only when it is about to execute `Deploy`, it will evaluate `HasChanges` and decide to skip or execute the target. 

There's a caveat however. Even if `HasChanges` turns out to be `false`, it will still execute `Prepare` and `DetermineChanges`, and since `Prepare` has a required parameter, it'll still require it to contain a non-`null` value. In other words, `OnlyWhenDynamic` only affects the current target and not any of its dependencies. So let's rewrite the target and use `OnlyWhenStatic` instead:

```csharp
    Target Deploy => _ => _
        .DependsOn(Prepare)
        .DependsOn(DetermineChanges)
        .OnlyWhenStatic(() => HasChanges)
        .Executes(() =>
        {
        });
```

The big difference with the previous example, is that Nuke will now evaluate `HasChanges` while it builds the dependency graph of the targets, _before_ `DetermineChanges` is executed. In other words, if `HasChanges` is `false`, which it is by default, neither `Deploy`, nor its dependencies will be executed. And because of that, `HasChanges` will never get reevaluated. As that would not be very useful, to make this work with `HasChanges`, you still need to use `OnlyWhenDynamic` _and_ use it on all the targets that you want to conditionally run, including their dependencies.

But fortunately, there's a better solution: Make the `HasChanges` property do the work itself instead of relying on a target like `DetermineChanges`. Then you can rely on  `OnlyWhenStatic` and don't have the sprinkle your codebase with `OnlyWhenDynamic`. Just make sure you cache the result of that work, for example, like this:

```csharp
    bool? hasChangesState = null;

    bool HasChanges
    {
        get
        {
            if (!hasChangesState.HasValue)
            {
                hasChangesState = // execute some run-time logic to set this field
            }

            return hasChangesState.Value;
        }
    }

    Target TargetUsingDynamicProperty => _ => _
        .DependsOn(Prepare)
        .DependsOn(DetermineChanges)
        .OnlyWhenStatic(() => HasChanges)
        .Executes(() =>
        {

        });
```

If only I realized that earlier...

## About me

I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 28 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


