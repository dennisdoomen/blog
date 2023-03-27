---

title: "What it takes to make legacy code bases rock solid"
excerpt: "Another thing I realized last year is that I really love working on legacy code bases and making them rock solid."

tags:
- legacy
- maintenance
- greenfield
- brownfield
---

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/ruins.jpg" class="align-center"/> 

Another thing I realized last year is that next to building green-field projects (who doesn't like that?), I really love working on legacy code bases and making them rock solid. It's so rewarding to take something that was in production successfully but is suffering from all kinds of technical issues (and the resulting team productivity issues) and make it better, release by release. 

Things (in no particular order) that I typically spend time on (although it all highly depends on the state, the goals, the needs, the culture, etc.):

* Talk to the people to understand the biggest pain points, both from a user perspective, product management perspective and developer perspective
* Establish principles and conventions for coding, naming, testing, design, documentation, source control and refactoring principles
* If there's no test coverage, introduce Characteristic Tests as a temporary safety net and then gradually introduce appropriately scoped automated tests. 
* Refactor code and create architectural and design-level seams to make it more testable and allow more isolation in the team
* Introducing a clean build pipeline so what you build and deploy locally as you would in production
* Setting up a clear branching and merging strategy
* Create an architecture poster to make sure everybody has the same understanding of the layers, modules, slices, etc.
* Group-review sessions to look at the legacy code and demonstrate/validate improvement ideas
* Look at the planning, refinement and estimation process
* Look at the tooling to see how they contribute or impede developer productivity

What I don't like doing? Research projects that never go into production…

So what about you? Do you prefer greenfield projects or digging your hands in an existing codebase. Let me know by commenting below. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).
