---

title: ".NET solution templates for class library packages"
excerpt: "A new open-source project for making it trivial to build high-quality NuGet packages with all the bells and whistles"
tags:
---

## About this new project

I like to build my software systems in a nicely broken down set of libraries that are easy to maintain, test and deploy based on the [Principles Of Successful Package Management](https://www.dennisdoomen.com/2016/10/principles-for-successful-package.html). But every time I or the teams I work with need to start a new library or reusable component, we have to scramble so much from other projects, I felt this good fill the gap.

So I've started to create a bunch of `dotnet new` templates to quickly get you started building high-quality libraries including everything you need to publish it on NuGet or make it available as open-source.

It includes:
* Multi-targeting to cover as many .NET frameworks as possible
* Code coverage using [Coverlet](https://github.com/coverlet-coverage/coverlet) and [Coveralls.io](https://coveralls.io/)
* Static code analysis using Roslyn analyzers [StyleCopAnalyzers](https://github.com/DotNetAnalyzers/StyleCopAnalyzers), [Roslynator](https://github.com/dotnet/roslynator), [CSharpGuidelinesAnalyzer](https://github.com/bkoelman/CSharpGuidelinesAnalyzer) and [Meziantou](https://github.com/meziantou/Meziantou.Framework).
* Auto-formatting using `.editorconfig` and settings honored by [JetBrains Rider](https://www.jetbrains.com/rider/) and [ReSharper](https://www.jetbrains.com/resharper/)
* A [Nuke](https://nuke.build/) C# build script that you can run locally as well as in your CI/CD pipeline
* A GitHub Actions workflow that builds, tests, packages and publishes your library
* An extensive read-me
* Automatic versioning using [GitVersion](https://gitversion.net/) and tagging
* Contribution guidelines
* Customized release notes templates for GitHub connected to pull requests labels.
* A test project using [xUnit](https://xunit.net/) and [Fluent Assertions 7](https://fluentassertions.com/)
* Validation of the public API of the library against snapshots using [Verify](https://github.com/VerifyTests/Verify)

This is the result of years of experience in building in-house and open-source libraries that are used by thousands of developers around the world. I hope it's a great starting point for building your own libraries. 

You can also use this as a starting point for a [GitHub Template Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository), fork and adapt the repository for your own company, or use it as an organization template in Azure DevOps.

## How to get started

Everything is documented at [the repository](https://github.com/dennisdoomen/dotnet-package-templates), but here's a summary of the approach:

1. Create a new directory for your library initialized with Git
2. Run `dotnet new class-library-package-solution -name TheNameOfYourAwesomeLibrary`
3. Make the necessary changes to the generated code 
4. Commit the changes to your repository into a new commit
5. Run `build.ps1` to build the code, run the tests, and package the library into a NuGet package in the `Artifacts` directory.
6. Use a Github Actions workflow to push to NuGet

Stuff you can customize includes

* Update the `README.md` with information about your library
* Review the guidelines in `CONTRIBUTION.md` to see if it aligns with how you want to handle contributions
* Adjust the .NET frameworks this library should target
* Adjust the namespace 
* Set-up labels in GitHub matching those in the `release.yml` so you can label pull requests accordingly
* Alter the coverage service that is being used.
* Determine if you want to use API verification against snapshots
* Study the Nuke `build.cs` file or invoking it through `build.ps1 -plan` to see how it works
* See if all dependencies are up-to-date
* Adjust the `funding.yml` to allow people to sponsor your project

## About me

I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 28 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


