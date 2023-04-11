---

title: "6 signals that your architecture is not visible enough"
excerpt: "I've been reflecting on common issues that make it hard for the developers to understand the architecture"

tags:
- solid
- architecture
- best-practices
---

With more than 26 years of experience, as a consultant, I help organizations in the .NET space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I'll share some of the most common pain points I run into. 

In my [last post](/2023/03/coding-smells) in this series, I've been diving into the depths of coding practices that have a smell. This time, I'd like to focus on the availability and discoverability of technical information about architecture. Some teams are more then willing to write documentation, but what's lacking is clarity on where to find that documentation, how it relates to the architecture and how up-to-date it still is. Here are some consequences of that.
 
* It is often not obvious where in the system some piece of functionality is implemented. This is particularly an issue when the code is structured along the technical aspects of the system instead of aligning with functional modules. This is frequently caused by developers that don't understand the internal boundaries and architectural seams of the system, either because they are too focused on the details, or because the architecture itself has been obfuscated.

* And if you look at the architecture from the code's perspective, you should be able to deduce the architecture. In fact, in an ideal world, the code should make the architecture evident just by looking at the folder names. The industry likes to call that a _screaming architecture_. Even if you decide to look at the system from the UI's perspective, you should get a decent understanding of where certain sections of the UI are implemented. And it shouldn't matter whether it's a desktop application or a modern single-page web application.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/architecture-visibility.png" class="align-center"/> 

* Even if a developer found a suitable place for some new piece of code, I've noticed that they aren't always aware how that place fits in the bigger picture. And if you don't know that, then you probably also don't know how that place relates to other parts of the system and what kind of coupling is allowed or not. Having some kind of visual representation can really make a difference. Unfortunately, developers that are very experienced with a code base often built a mental map of the architecture and know their way instinctively. Or worse, they may reject the need for such a visual representation because it may surface the inconsistency of the code base. All issues that don't help protect the internal architectural boundaries.

* It doesn't help if documentation is not up-to-date, especially if it is supposed to cover the architecture and its underlying principles. Some organizations ensure writing documentation is part of the Definition of Done. That's a good thing, but doesn't guarantee that you'll end up with useful documentation. Documentation needs to have purpose, scope, and more importantly, the right audience and a clear actuality. It should be completely obvious where to find it and how to navigate through related content. And all of this also applies to technical and architectural documentation. That's one of the main reasons why I don't believe in UML tools like Sparx Enterprise Architect. Architecture diagrams should only exist as visual support to tell a story, even those using the [C4 Model](https://c4model.com/).

* Now imagine a beautiful world where the architecture, the code and the documentation are all nicely aligned. Unfortunately (or fortunately) things change. New functionality, new environments, new technical insights and refactoring needs are all inevitable in a successful system. And that's fine, provided that the developers involved keep the rest of the developers and stakeholders posted about those changes. In a previous post, I've mentioned internal blog posts already, but technical sessions, architecture decision logs and similar techniques seem to be a bridge too far for many teams. 

* A small, but not unimportant part of the architecture is the code itself. Does every code project (such as a Github or AZDO repository) a clear read-me explaining the purpose of the project, how it is released, who owns it, where to find the build artifacts, what you need to compile the project and how it is deployed in production. I've seen too many projects, both within companies as well as in the open-source world, that lack that kind of information.

Those were only six points. I've seen quite some teams that ran into structural problems because they were convinced that "the code is documentation enough" or "I know the architecture by heart". And it really isn't that hard to identify the necessary ingredients to make people aware of the architecture, both in code as well as at the documentation level. 
	
Do you recognize one or more of these signals? Do you think they apply to your organization? Let me know by commenting below.

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).