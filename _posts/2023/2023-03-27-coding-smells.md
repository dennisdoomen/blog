---

title: "9 coding practices that have a smell"
excerpt: "Naming in software is hard, so here's how I name and group my automated tests to use them as documentation'"

tags:
- naming
- unit-testing
- test-driven-development
---

With more than 26 years of experience, as a consultant, I help organizations in the .NET space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I'll share some of the most common pain points I run into. 

In the [last post](/2022/01/symptoms-traceability) of this series, we looked at both the traceability of technical and architectural decisions as well as the micro-decisions that were made at the code-level and (hopefully) captured in your source control system. So the logical follow-up to that is to look at the readability and maintainability of the code itself. There's a lot to cover there, so let's see what I tend to run into.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/code-smells.png" class="align-center"/> 

* Code that may be readable but which purpose is unclear doesn't make anybody happy. If it seems to have a bug or something needs to change functionally, being able to understand the original intention of the developer that wrote it is going to be crucial. And that doesn't necessarily mean that that's what the code is doing right now. The issues usually come from improper naming, the way the code is organized and the lack of functional documentation at the code level. Especially that last category is often difficult. Either it's too technical or repeats unnecessary context.

* It becomes even more a challenge if there's a bug in the automated tests itself. When a test fails, your first thought is usually that the code-under-test must behave incorrectly. But I'm not making things up if I say that I've encountered quite some tests cases that were testing the wrong thing. That's why it's so important to treat test code as real code including a crystal clear intention.

* Talking about intention, another issue I see a lot is that code is not written in such a way that it clarifies the algorithm that a method tries to execute. Methods, functions and operations are ways to encapsulate complexity and provide an abstraction that makes it easy to work with. But as soon as you want to understand its behavior, it's going to be important to "see" the algorithm. Mixing invocations to other methods and low-level statements isn't helping with that.  

* Code documentation is a sensitive topic as well and often a source of passionate debates. I generally see a lot of extremes here. On one side you'll find code bases that have no code-level documentation at all. Strange arguments I'm often hearing are "code _is_ the documentation" or "when the code changes, I have to rewrite the documentation". But on the other side where people dogmatically require all code to be documented isn't the right way either. Just like everything else, you'll need to find the right balance. Understanding the conceptional difference between intention and implementation details is an important capability here.

* An issue that's much simpler to fix is when developers force you to scroll up and down the file just to understand the flow of the code. That's like ordering the paragraphs of a page in a novel in some random order. And yes, I know that analogy isn't 100% perfect, but I'm sure you get my point

* Another practice that has both lovers and haters is refactoring. I've seen teams where refactoring is always under pressure from being abandoned because developers have to request permission from their product owner or manager before doing any kind of refactoring. And even though I strongly disagree with that, I've seen developers misunderstanding the difference between refactoring and redesign. The former should be part of their day-to-day work, but the latter is something that needs to be planned. So you can imagine this doesn't help establish trust between developers and management. 

* With respect to class design I've seen my fair share of bad examples. Having a derived class that only makes sense after you jump around through multiple layers of base and derived classes is one of those. But equally worrisome are classes with names like `Manager` or `Helper`. I see those as an excuse to have some place to group all kinds of unrelated technical functionality. And what about all these "senior" developers who love to introduce layers, abstractions and design patterns "because that's SOLID" or "to be prepared for the future"? You'll immediately recognize those by looking at the number of interfaces which name is the implementing class prefixed with `I`.

* And it doesn't stop with design. I've seen plenty of code smells that make me frown. Think about deeply nested structures, often found in very long methods, full with "magic numbers" and variables with cryptic names. And if you're unlucky, and somebody has been dogmatically using `var` in C#, those cryptic names are the only thing you have. And what about boolean parameters that make it impossible to understand their purpose, e.g. what does `true` or `false` really mean? And don't forget the need for multiple dots to de-reference a deeply nested property or field. That's a sign of bad design. 

* Are you using [SonarQube](https://www.sonarsource.com/products/sonarqube/) to automatically verify your C#, TypeScript or JavaScript code against best practices from the industry? And if so, did you go beyond the default rule set or did you just disable most of the rules? And what about using [ESLint](https://eslint.org/) for JavaScript/TypeScript? And StyleCop/FxCop or the [free Roslyn](https://github.com/bkoelman/CSharpGuidelinesAnalyzer) analyzers for C#? Do you use an IDE that [understands those](https://www.jetbrains.com/rider/), provides automatic formatting based on `.editorconfig` or a [Prettier](https://prettier.io/) configuration, and may even help you write clean code with its out-of-the-box features? Considering the many tools to our disposal that can detect code smells, design smells and even architectural smells, it's a surprise developers still produce so many of them. 

Those are just a few of the many, sometimes dogmatic, best practices from our industry and a lot has been said and be written about this. Finding a good trade-off is hard, in particular because of the many opinions that senior developers often have. Waving with another book isn't helping there, so I totally get that these are sensitive topics within any organization.

Do you recognize any of these smells? What do you do to mitigate them? And what about your own smells? Care to share them here? Let me know by commenting below.

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).