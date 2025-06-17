---
title: Kickstarting your libraries with the .NET Library Starter Kit
excerpt: A battle-tested starter kit for building open-source and internal NuGet libraries, born from half a billion downloads.
author: Dennis Doomen
tags: [.NET, Open Source, Templates, Libraries, NuGet, DevOps]
---

While starting a bunch of new open-source projects, I found myself repeatedly copy-pasting boilerplate code from [FluentAssertions](https://fluentassertions.com/), a (partially) open-source project with over half a billion downloads. That made me realize there must be a smarter way to encapsulate all that knowledge and experience into a reusable format.

That’s how the [.NET Library Starter Kit](https://github.com/dennisdoomen/dotnet-library-starter-kit) was born: a set of `dotnet new` templates to quickly get you started building high-quality binary or source-only libraries, including everything you need to publish them on NuGet or make them available through Inner Sourcing.

## Multi-targeting
By default, the templates target .NET Standard 2.0 and 2.1, .NET Framework 4.7, and .NET 8. This gives your library maximum reach, including compatibility with Xamarin, Mono, Universal Windows Platform (UWP), and Unity. Of course, if you don't need that level of compatibility, you can easily adjust the targets.

## Separate templates for in-house and open-source libraries
Not every library is meant to be open-source, so the starter kit offers separate solution templates for both internal and public-facing libraries. And not just for GitHub, but also for Azure DevOps—albeit only for internal use. So even developers [still stuck on AZDO](https://www.dennisdoomen.com/2024/01/github-vs-azdo.html) have an excuse to skip Inner Sourcing.

## Binary or source-only packages
Everyone’s familiar with traditional NuGet packages that contain DLLs and EXEs. But these can introduce diamond dependency issues if different packages depend on conflicting versions of shared libraries. Source-only packages (also known as content-only) solve this by shipping the original C# source files. When added to a project, the files are compiled into your assembly as if they were part of your own code. It's ideal for small utility libraries like [Reflectify](https://github.com/dennisdoomen/reflectify) and [Pathy](https://github.com/dennisdoomen/pathy).

## Code coverage using Coverlet and Coveralls.io
Even though many managers treat code coverage as a KPI, it still holds value. Since different teams use different tools to report coverage, the starter kit integrates with [Coveralls.io](https://coveralls.io/), a free service suitable for open-source projects. For internal libraries, you can tweak the included build script to make it work with other platforms.

## Auto-formatting with special support for JetBrains Rider and ReSharper
With these templates, you don't have to waste time debating code layout in pull requests. We include a preconfigured, slightly opinionated `.editorconfig` to get you started. For [JetBrains Rider](https://www.jetbrains.com/rider/) and [ReSharper](https://www.jetbrains.com/resharper/), the setup includes `.sln.dotsettings` that lights up a “Safe Clean-up” [auto-formatting profile](https://www.jetbrains.com/help/resharper/Code_Cleanup__Index.html). It enables consistent formatting and C# modernization with a single shortcut.

## Static code analysis using Roslyn analyzers
The template includes several high-quality open-source Roslyn analyzers to prevent common mistakes:

- [StyleCop Analyzer](https://github.com/DotNetAnalyzers/StyleCopAnalyzers) for enforcing code style conventions.
- [Microsoft.CodeAnalysis.BannedApiAnalyzers](https://github.com/dotnet/roslyn-analyzers) to restrict certain APIs via a `BannedSymbols.txt`. Example [here](https://github.com/fluentassertions/fluentassertions/blob/main/Src/FluentAssertions/BannedSymbols.txt).
- [Roslynator.Analyzers](https://github.com/dotnet/roslynator) and [Meziantou.Analyzer](https://github.com/meziantou/Meziantou.Analyzer) for code quality and performance.
- [CSharp Guidelines Analyzer](https://github.com/bkoelman/CSharpGuidelinesAnalyzer), which complements the [Aviva Solutions C# Coding Guidelines](https://csharpcodingguidelines.com/).

These analyzers are enabled through `Directory.build.props` and run only on the .NET 8 target to keep compile times reasonable. Rules are preconfigured in `.editorconfig` with sensible defaults for most teams.

## A C# build script that supports local runs, debugging, and refactoring
[Nuke](https://nuke.build/) is one of those rare open-source gems that make a real difference. The template includes a Nuke-powered C# build script that you can debug, refactor, and run locally. Only a minimal amount of YAML is needed to integrate with GitHub Actions or Azure Pipelines. Say goodbye to brittle CI scripts and trial-and-error debugging.

## A beautiful README to get you started
A good README is essential, whether you're publishing to NuGet or promoting Inner Sourcing. The included template covers it all: purpose, getting started, maintainers, support, build instructions, links to documentation, and credits to dependencies and contributors.

## A test project that emphasizes the specification aspect
We’re big proponents of Test-Driven Development. The templates include a default test project with the `Specs` postfix. This naming convention helps reinforce the idea that your tests are specifications of behavior—not just verification code.

## Validation of the public API using Verify
The template adds an `ApiVerificationTests` project that uses [Verify](https://github.com/VerifyTests/Verify) to detect breaking changes in your public API. If you're okay with a change, just accept it using `AcceptApiChanges.ps1` or `AcceptApiChanges.sh`. Or use the [Verify Support](https://plugins.jetbrains.com/plugin/17240-verify-support) Rider plugin for an even smoother experience.

## NuGet auditing and license scanning
The template has NuGet vulnerability auditing enabled out of the box. You can learn how to configure it to your needs [here](https://github.com/dennisdoomen/dotnet-library-starter-kit?tab=readme-ov-file#about-nuget-auditing).

It also includes [PackageGuard](https://github.com/dennisdoomen/packageguard), a lightweight open-source tool integrated into the Nuke build to scan licenses and enforce dependency policies. It’s a free and flexible alternative to pricey enterprise tools.

## Automatic versioning using GitVersion and tagging
Versioning your libraries manually is error-prone and hard to scale. The templates use [GitVersion](https://gitversion.net/) to automatically calculate semantic versions based on your Git history. It supports branch-based workflows and ensures consistency across releases. Combined with Nuke, you’ll get automatic version bumping and tagging with every successful release pipeline.

## Contribution guidelines with learnings from 15 years of OSS development
Open-source thrives on good collaboration. The templates include a `CONTRIBUTING.md` file based on more than a decade of community work. It includes best practices for pull requests, branching strategies, commit message conventions, and links to your project's code of conduct.

## GitHub issue templates
You'll also get preconfigured GitHub issue templates for reporting bugs, requesting features, or submitting questions, all inspired by many years of dealing issues in FluentAssertions. These help set expectations and ensure maintainers get the context they need from contributors right away.

## Customized release notes templates tied to pull request labels
Release notes can be painful to maintain manually. The templates include GitHub release note templates connected to pull request labels. These labels drive automatic changelog generation based on the nature of the PR—feature, fix, chore, breaking change, etc.

## Forkable
Since the starter kit is open source, you can easily fork it, customize it for your organization, and publish it as a [GitHub repository template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template) or use it manually within Azure DevOps. That makes it easy to enforce standards and conventions across all your internal teams.

# About Me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 29 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), [PackageGuard](https://github.com/dennisdoomen/packageguard), [Pathy](https://github.com/dennisdoomen/pathy), [Reflectify](https://github.com/dennisdoomen/reflectify), the [.NET Library Starter Kit](https://github.com/dennisdoomen/dotnet-library-starter-kit) and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me through [Email](mailto:dennis.doomen@avivasolutions.nl), [Bluesky](https://bsky.app/profile/dennisdoomen.com), [Twitter/X](https://twitter.com/ddoomen) or [Mastadon](https://mastodon.social/@ddoomen)
