---

title: "8 coding lessons from almost three decades of software development"
excerpt: "Build better code, one habit at a time and other practical advice to enhance your team’s practices and your future self’s sanity."

tags:
- CleanCode
- CodeReviews
- SoftwareDevelopmentPractices
- MaintainableCode
- DevTeamCollaboration
---

## 1. Write code for your colleagues (or your future self)

I still remember the days when I challenged myself to write code that did as much as possible in as few lines as possible. I thought this was impressive and demonstrated my skills as a professional developer. How wrong and naive I was. These days, I know that I should write code that is easy for my colleagues to understand, and that colleague might be me, six months from now. Such code should be intention revealing, use clear naming, be properly structured, and, if necessary, include documentation. In other words, it should be "clean" code that looks like it was written by someone who cares. How do you tell? Just ask your colleagues...

## 2. Superficial code reviews are as bad as no reviews 
Code reviews have a lot of value:

* Find common mistakes 
* Find inconsistencies with similar solutions elsewhere in de code base
* Identify new or updated NuGet or NPM packages that don't meet the standards 
* Identify unwanted coupling
* Identify constructs that are non-trivial to other developers (which can be you in the future) 
* Find potential refactoring opportunities 
* Make sure the rationale for changes is clear 
* Make sure that the real changes are not hidden between mass refactorings

So take them seriously. They will help you and your team both now _and_ in the future. And don't forget the opportunity it provides to learn from each other. If the only you're doing is marking the pull request as "approved" with a comment like "Looks good to me", then don't bother at all.

## 3. Make code reviews a worthwhile and fun exercise

Assuming you agree with my earlier point about the importance of code reviews, you'll likely also agree that it's equally important to provide constructive, useful, and unambiguous comments. You need to make it clear whether something is a dealbreaker for you, just a suggestion, an opportunity for refactoring, or something that requires further discussion.

And it's not just to help younger developers who might look up to you and can't yet make those distinctions. It's also a way to force yourself to consider how important a comment is and whether it might be better to omit it altogether.

That's why I prefer to use [emojis](https://github.com/erikthedeveloper/code-review-emoji-guide) to help clarify my expectations for a comment and prevent myself from nitpicking or imposing my preferences on others. For example, I use 🤔 to trigger a discussion on a suggestion or an alternative solution. If I want to plant a thought for later, I might use a 🌱. Refactoring opportunities are prefixed by ♻️, and ideas for leaving the codebase in a slightly better state are marked with 🏕️. Yes, it's a bit silly, but it also makes reviewing fun again.

And next to that, make sure you use a source control system that allows you to group review comments, request re-reviews, and review individual commits. GitHub and GitLab can do that, but Azure DevOps [definitely not]({% post_url /2024/2024-01-07-github-vs-azdo %}).
 
## 4. Never ignore errors or warnings, even if you know they are expected or safe

You’d be surprised about how many codebases I've seen where the build and deployment pipeline is full of warnings. Or what about database scripts that fail on existing tables or columns, raising all kinds of ORA errors (yes, I'm looking at you, Oracle)? And don't forget log files full of stack traces and warnings.

Never, ever accept this. If there are too many, people start to ignore them. Treat compiler warnings as errors that should fail the build, and treat database script issues as fatal errors that need smarter DDL statements. Finally, be very specific about what a warning in a log file means, when to raise it, how to handle transient problems, and ensure every error or stack trace is thoroughly analyzed by a developer.

## 5. Let tooling handle the coding style debates

Over the years, I've seen my fair share of heated debates about coding conventions—how to layout code, the maximum length of a line, the spacing between braces and parentheses, when to use a line break, and many more details like that. I've also been guilty of spending time discussing these conventions in codereview comments, which obviously didn’t help foster a positive environment. But don’t get me wrong, I still care about them. It’s just that I’ve learned to rely on tooling to handle formatting.
 
You can use an `.editorconfig` file to capture many of the layout rules, and most editors will honor them. If your code contains JavaScript or TypeScript, then an `.eslintrc` file can take care of those rules. Additionally, if you’re working in the C# space and use Roslyn analyzers or tools like StyleCop, you can include their configuration in the `.editorconfig` as well. But it doesn’t stop there—[JetBrains Rider](https://www.jetbrains.com/rider/) and [ReSharper](https://www.jetbrains.com/resharper/) offer plenty of options to fine-tune the layout or customize how you use newer C# features, and those can be added to the `.editorconfig` too. Lastly, every JetBrains IDE supports custom [clean-up profiles](https://www.jetbrains.com/help/rider/Settings_Code_Cleanup.html#cleanup_tasks_for_selected_profile) that can reformat and optimize files, folders, and entire projects based on your preferred settings. And yes, all of this can be stored in source control via the `.dotsettings` file so your whole team can benefit.

## 6. Adopt the principles that make open-source projects successful

I've been an open-source developer for [a long time now](https://github.com/dennisdoomen), but only in recent years have I realized that many of the practices we use in open-source are just as applicable in traditional development teams. Unless your entire company consists of just a handful of developers, you're inevitably going to face team scalability challenges.

A great solution to this is breaking down your codebases into separate repositories that are individually releasable—provided you carefully design the dependencies between them. Even if you're not splitting your codebase for scalability, there will always be components that you may want to share with other teams.
 
I'm a big proponent of treating internal code the same way open-source developers do, often called _inner sourcing_. This means anyone should be able to fork a repository, suggest improvements, and feel confident and informed enough to understand what's expected. 

To achieve this, the repository should have a comprehensive read-me, high-quality code with static analysis, clear contribution guidelines, thorough documentation, proper test coverage, a well-defined branching strategy, [Semantic Versioning](https://semver.org/), and a build pipeline that can be [tested and adopted locally]({% post_url /2020/2020-03-01-reasons-for-adopting-nuke %}). The service hosting the source code should support forks without requiring write access and provide extensive code review features. GitHub is [obviously]({% post_url /2024/2024-01-07-github-vs-azdo %}) ideal for this. In short, we should apply the same principles that make open-source projects successful.

## 7. Leave the campground cleaner than you found it

Nobody's code is perfect—not mine, not yours, not even the code written by the best software developer in the world. We make decisions based on what we know at the time: the libraries we’re familiar with, the programming languages we’re comfortable with, and the pressures we face within our teams. Over time, circumstances change, and those decisions might need to be revisited.

Maybe the library you're using is outdated. Perhaps the code you wrote was altered by someone who didn’t fully understand its original purpose. The naming might no longer feel intention-revealing, or a better pattern or construct could make the code more maintainable. The list goes on. None of these issues arise from malice or incompetence—they’re simply part of the job. In fact, if you’re not facing these challenges, it’s likely your codebase isn’t being used anymore. Even feature-complete code isn’t immune to the effects of time and change.

In a way, you could say that code "rots" if you don’t take care of it. That’s why you should make it a habit to look for refactoring opportunities whenever you modify code. This is the perfect moment to improve it since you’ll already be in context, trying to understand it and implement changes. But don’t go overboard. I’ve seen developers become overzealous, spending more time on refactoring than delivering features. Sometimes that’s necessary, but more often, I like to follow the Campground Rule: “Leave the campground cleaner than you found it.” In other words, ensure every commit makes the code a little better. That’s how you stay on top of it without losing focus on your goals.

## 8. Development practices really reinforce each other

Over the course of our careers, we are trained to adopt numerous principles. I'm not just talking about concepts like Clean Code, SOLID, Test-Driven Development, DRY, KISS, and other acronyms, but also practices like breaking down tasks, keeping Pull Requests reviewable, and [avoiding obfuscation]({% post_url /2020/2020-03-18-keep-source-control-history-clean %}) in your source control history. There's so much to learn as a developer, and the risk of becoming dogmatic about these practices is very real.

I regularly run full-day workshops on writing maintainable and testable code. I can't pinpoint exactly when it happened, but at some point, I realized how many of these practices actually reinforce one another. For instance, creating an estimate can guide a task breakdown, which opens up opportunities for swarming—where multiple developers collaborate to complete work. It also enables smaller pull requests, which are easier to review thoroughly and quickly. Similarly, pull requests with small, focused commits not only speed up reviews but also contribute to a cleaner source control history.

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 28 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


