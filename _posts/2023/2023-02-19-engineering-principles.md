---

title: "How modern engineering principles reinforce each other"
excerpt: "How modern engineering practices contribute to readability, testability, isolation, tracebility and discoverability."

tags:
- maintainability
- maturity
- codingpractices

---

[Last time](https://www.continuousimprover.com/2023/02/maintainable-code.html), I tried to make a case on why I believe readability, testability, isolation, tracebility and discoverability should be the foundations for writing testable and maintainable code. Another perspective,one that I often use in my workshops, is to show how modern engineering practices contribute to those five aspects of our profession. Here's a map of those principles and how they reinforce each other. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/engineering-principles.jpg" class="align-center"/> 

A lot of developers don't realize how these practices all fit together. So let's look at an example from the diagram below, the speed and thorougness of code reviews. 

It all starts with the estimation process, which, to be completely open, I don't care about. They will be wrong anyway. But the mere process of trying to come up with an estimate will force you to breakdown the work in bite-sized chunks your brain can assign an estimate to. A good breakdown has two big advantages. It creates opportunities for the team to swarm on the same work and thereby, for once, finish work before the end of the iteration ends. But it also increases the chance you end up with a pull request consisting of small and focussed commits. 

And such a pull request doesn't require a lot of time to review, does it? And I'm sure it will be reviewed much more thoroughly. Just ask yourself, what do you do when you're asked to review a pull request with hundreds of files? Glance over the changes and then approve it without further comments? Yes you do. But a PR that contains a small number of changes, which code has been written in an elegant, mostly self-documented way, _and_ is covered by automated tests is much more fun to review, right? And that's not all. Those small and focussed pull requests will contribute to a clean and tracable soure control history as well. 

Another, less complicated example is the way refactoring contributes to all five pillars I started this post with. Making it a habit to continuously refactor your code will obviously improve the readability of both your production code as well as your tests. But it will also help you better understand the natural seams in your system and the logical place where things belong. Part of the refactoring effort is often to move code that doesn't belong at some place to its correct location. And that will eventually contribute to the discoverability of your codebase. 

But finding the right level of refactoring is hard. If you don't spend enough time on it, your code rots and your team lead will be pissed off. But if you spend too much time on it, your regular work derails and your product owner or team lead will be pissed off as well. That's why I love the article [Natural Cause of Refactoring](https://www.infoq.com/articles/natural-course-refactoring/). It provides a nice practical approach where inline comments like `// NOTE`, `// SMELL` and `// REFACTOR` are used as breadcrumbs for a future you. 

Assuming the rest of the practices are self-explanatory, what do you think about my theory? Does it make sense to you? Any thoughts? Let me know by commenting below. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).
