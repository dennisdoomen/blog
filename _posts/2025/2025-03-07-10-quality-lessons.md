---

title: "10 quality lessons from almost three decades of software development"
excerpt: "About testability, code reviews, flakiness, maintainability, pipelines and source control history"

tags:
- CleanCode
- CodeReviews
- SoftwareDevelopmentPractices
- MaintainableCode
- Quality
---

## 1. Understand the difference between internal and external quality

One of the most challenging parts of our job as developers and architects is helping managers and non-technical product owners understand what we mean by bad software quality. For much of my career, I struggled with this. That is, until I realized the key was distinguishing between internal and external quality.

External quality is what managers and product owners care about. They’ll notice when responsiveness or usability is poor, or when bugs—especially recurring ones—pile up. But internal quality? That’s our domain. It's the part they don’t see, until #technicaldebt slows down feature delivery to a crawl.

To bridge this gap, I started framing internal quality issues in terms they could relate to—specifically by using five "ilities" that matter most:
 
* **Readability**; What does the code do, and how does it work?
* **Testability**; Does it do what it’s supposed to?
* **Isolation**; Can I change this without breaking something else?
* **Discoverability**; Where in the system is this behavior defined?
* **Traceability**; Why was this decision made (in the code, architecture, or design)?

Presenting internal quality through these lenses has made my concerns much clearer to product owners. Maybe this approach will help you too.

## 2. Make your code automatically testable

Do you know that warm, fuzzy feeling when you're modifying code in a codebase you're not 100% familiar with, but you're totally confident the automated tests have your back? Feels great, right? Now, hold onto that thought and imagine a similar codebase—but without that kind of test coverage. Sounds terrifying, doesn't it

Now, what if you do know that codebase inside and out? Are you confident enough to make changes without risking a catastrophic bug? Really?

I've been maintaining an open-source project for 15 years, and let me tell you — I’m absolutely, positively, 100% not confident doing that without my tests.

For those who don’t feel confident, it’s time to invest in automated testing. Start by building a temporary safety net using Michael Feathers' [Characterization Test](https://michaelfeathers.silvrback.com/characterization-testing) strategy. Then, begin redesigning your code to support a test-first approach. Over time, you’ll move toward a Test-Driven Development (TDD) workflow, writing tests upfront.

And one last thing: treat your tests as first-class citizens. A codebase where 40–50% of the code is dedicated to tests? Totally healthy.

## 3. Don't accept flakiness in automated testing

Every developer who has tried to build a suite of automated tests for a browser has faced the nightmare of flaky tests. Sure, you might feel proud of having a solid amount of automated testing in place. But if you can't rely on those tests due to flakiness, it can become a major headache. Determining whether a test failed due to an actual issue in your codebase or simply because of flakiness is challenging—especially when the failure isn’t tied to a specific code change.
 
More often than not, people just re-run the test and hope for the best, particularly if it’s long-running. Assigning someone to investigate these failures is difficult, and the uncertainty can derail productivity.
It may sound obvious, but don’t treat flakiness as technical debt to be addressed later. Treat it like a production failure that requires immediate attention. 

If you’re lucky enough to use a build tool like [JetBrains TeamCity](https://www.jetbrains.com/teamcity/), you can track test flakiness over time, mark tests as flaky, and mute them from the build pipeline. Better yet, assign the flaky test to someone for analysis and fix it while allowing the build to succeed. This prevents multiple developers from wasting time trying to figure out if they need to act on a flaky test. If you don’t have these tools, agree as a team to prioritize fixing flaky tests to get the build green again.

## 4. Write code that you can change with confidence

I once asked "the internet" what maintainable code is all about. Some of the responses included answers like code that shows somebody cares or code that doesn’t need explanation, just like a good joke.

But the most important aspect of maintainable code—one most people agreed on—is the ability to change it with confidence: confidence that you understood it, made the right changes, and didn’t break anything else.

Many books have been written on this topic, and there are hundreds of opinionated rules about it. But here are a few guidelines that have helped me:
 
* ✅ Code should read like a book—members are ordered by execution order, not visibility.
* ✅ All code in a method or function should be at the same level of abstraction.
* ✅ Members should try to stay within 15 lines of code.
* ✅ Member names should be functional and explain their purpose, not their implementation.
* ✅ Code documentation should describe why a member exists, not how it works.
* ✅ Inline comments are fine if they provide extra context.
* ✅ Either return immediately at the beginning or all the way at the end, but not in the middle.
* ✅ Avoid boolean parameters—use an enum or separate methods instead.
* ✅ Don’t introduce abstractions just for dependency injection.
* ✅ Group code by functionality rather than technical structure for better clarity.
* ✅ Ensure sufficient test coverage: 80% is nice, 90% is great, 95% is too much.
* ✅ Don’t test too small; test internal boundaries, but not the implementation details inside them.

What's your number one rule to keep code maintainable?

## 5. It never happens only once

Have you ever written a quick-and-dirty script, only to find it being reused over and over again? Or been asked to create a prototype or proof of concept (POC) that somehow made its way into production? And what about that weird bug that magically resolved itself during testing—only to resurface in production later? Issues like these don’t just disappear.

My advice? Unless you’re building an MVP, don’t accept them. And even if it is an MVP, make sure you allocate time to replace that one-off solution with a proper, long-term fix.

Don’t settle for ad-hoc solutions—whether it’s cleaning up a log directory, rerunning a flaky test, retrying to work around file locks, permission issues, or timeouts. Always address problems at their root.

Your future self will thank you.

## 6. Avoid any manual steps during deployment

The ability for a team to deploy a properly tested and versioned software system on their internal or cloud-based infrastructure is a good indication of the maturity of a team. Any manual step in that process (with the exception of manually triggering a deployment pipeline) is a potential liability and must be automated. People make mistakes, regardless of how meticulous they are. It's a fact of life. 
 
On the other hand, a well-designed deployment pipeline takes care of compiling the source code, invoking any codeanalysis tools, running the automatedtests, versioning the deployment artifacts, provisioning the infrastructure, updating the database schema, deploying to a staging slot and then, after proven healthy, swaps it with the production slot. All without having to share production passwords to developers, and without anybody connecting to the production environment and manually running scripts and other command-lines. And if your manager doesn't see the value, I'm pretty sure an audit related to the many security standards we have these days (ISO 27001, NIS, etc.) will help convince them.

## 7. Treat a build and deployment pipeline as you treat your code

A particularly painful experience I once had was dealing with flakiness in a deployment pipeline on Azure DevOps. I had to reproduce and fix it through trial and error. Since it was based on YAML and built-in tasks, the only way to troubleshoot was by making some modifications, queuing the pipeline, and waiting for the results. There’s simply no way to test this differently.

Beyond that, a build and deployment pipeline is much more than just a series of simple tasks. It evolves with the codebase, can become quite complex, and therefore must be easy to understand and refactor when needed. Too many companies try to stick with YAML pipelines, working around limitations by cramming PowerShell or Bash scripts alongside the YAML files - even though there are much better alternatives. This doesn't even take into account the potential need to migrate from one build engine to another, such as moving from Azure DevOps Pipeline to GitHub Actions, or, for a superior experience, adopting JetBrains' TeamCity.

Personally, I prefer to treat build and deployment scripts as part of the codebase and use a programming language I'm comfortable with. In the past, I’ve used PSake and PowerShell, but these days, [Nuke](https://nuke.build/) and C# are my go-to tools. This setup provides access to the navigation, refactoring, and debugging capabilities I’m used to, and it allows me to test the entire pipeline from my local development environment.

## 8. After 1.0 is shipped, be responsible

Being able to reuse functionality through #NuGet, #NPM, or other package managers has been one of the most successful mechanisms for promoting #reusability. In the .NET and TypeScript realms, we use packages everywhere, relying on versioning schemes to understand the impact of updates. I often frown upon packages that haven’t reached version 1.0 yet. Does this mean the author isn’t ready to commit to the current API and behavior? Should we expect #breakingchanges? And do the version numbers mean anything at all? Do they have a proper #releasestrategy?

After version 1.0, it’s time to be responsible. Your users or developers depend on stability. First, choose a release strategy — will you adopt regular releases, or only push new versions when necessary? A clear plan helps align teams and set expectations. Next, always apply [Semantic Versioning](https://semver.org/). It’s a simple system: increment the major version for breaking changes, the minor version for new features, and the patch version for bug fixes. Speaking of breaking changes, avoid them whenever possible. Introducing major changes can frustrate users, so provide clear deprecations and alternatives. Lastly, don’t forget documentation. Clearly document your code for both your team and your users. Well-documented code saves time, prevents errors, and fosters collaboration.

## 9. Cherish your source control history

Given what I previously said about capturing your decisions and how that affects the way you create your pull requests and commits, it should come as no surprise that I care deeply about the quality of #pullrequest and #commit messages. Imagine you're trying to understand why someone increased the timeout for executing a SQL query to 60 seconds. Nothing is more disappointing than finding a pull request or commit description that states something vague like, "Increased the SQL timeout". This type of description tells you what was done but fails to explain the problem the developer was solving. Similarly, a pull request containing a single commit that mixes refactorings with meaningful changes is likely to cause as much frustration as the previous example.
 
Since I often work on existing codebases and strive to truly understand why the code behaves a certain way or is written in a certain style, I rely heavily on a file’s history. Having felt the pain of not being able to do that, I go to great lengths to carefully organize changes into focused commits or even separate pull requests. I usually start with a single temporary commit, which I keep amending until I’m ready to prepare the pull request for review. I also make sure that every commit has a single purpose, with a title explaining that purpose and a description that summarizes the rationale behind the change.
 
This approach not only keeps your source control history useful but also makes it easier for colleagues to perform thorough reviews. By reviewing commit-by-commit, a colleague can quickly understand that all changes in a particular commit are related to a specific task, like a rename, saving significant time. Just make sure you don't squash those commits when merging.

You want to know how to do that? Check out [my previous post]({% post_url /2020/2020-03-18-keep-source-control-history-clean %}) on this.

## 10. Don't be afraid to reject a PR because it's too hard to review

Ever been asked to review a PR with 100+ files?

We've all been there. You scroll through the code, feel overwhelmed, and drop that classic "LGTM" comment. But here's the thing—I always reject these massive PRs. Why? Because a quality code review becomes impossible with that many changes.

My solution is simple:
* Ask developers to split changes into separate, focused commits.
* Even better, break them into multiple PRs.
* This is especially important for large-scale changes, like renaming variables or refactoring.

Pro tip: Using Azure DevOps? Separate PRs are a must -- its UI makes multi-commit reviews a nightmare! Or switch to GitHub. 

Remember, smaller PRs = better reviews = higher quality code

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 28 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


