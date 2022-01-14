---

title: 7 symptoms of insufficient traceability in your software project
excerpt: "Another part of what I call internal quality is the traceability of decisions, choices and considerations in software development.."

tags:
- software documentation
- source control
- traceability
- logging
- semantic versioning
---

With 25 years of experience, as a consultant, I help organizations in the .NET space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I'll share some of the most common pain points I run into.  

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2022/traceability.png" class="align-center"/> 

In the previous post, we looked at the testability of your system, an aspect of software engineering of which nobody will challenge the necessity. But another part of what I call internal quality is the traceability of decisions, choices and considerations. This is reflected in a variety of symptoms that I will discuss now.

* I sometimes ask a team about the rationale for an architectural or technical decision. You'll be amazed how often nobody remembers the reasons, even if the original developer is still available. And if they did capture that decision somewhere, it is often not clear which other options were considered and why the final option was ultimately chosen. A consequence of this is that often, technical choices are questioned over and over again, especially when new people join the team. If the thoughts behind that decision are no longer available, the entire process will start again.

* Insufficient communication of technical changes and their consequences on other teams through internal blog posts, notifications on internal communication tools such as Slack and Teams, technical sessions and group reviews. With small teams, it's easy to keep each other up to date. But as soon as your organization grows, you'll start to notice that information doesn't flow that freely anymore, resulting in people solving the same problems multiple times.

* Writing documentation without carefully thinking about who it is intended for and how actual it's gonna be. This often translates into documentation that is either too technical, too high level, or a weird mix of both. In any case, it won't work well for any of the intended audiences. Additionally, it is crucial to decide upfront whether it is a piece of text that is written once and has a natural decay, or whether it is something that needs to be updated continuously. And that also leads to choosing the right tool for that type of documentation. Not every tool works everywhere.

* Releasing software without making it completely obvious from the version number what this means for existing users? Is it immediately clear that it contains a bug fix that should be rolled out as soon as possible? Or does that release only contain new functionality that is totally safe to upgrade to? And even if there is already some kind of version numbering convention, I sometimes see that a so-called build number is attached to it. I understand why some teams do that, but producing different numbers for the same source code version is a red flag for me. There's no need for that.

* Code changes in your source control system that don't help you understand why the changes were necessary. In most code bases, you should be lucky if those explain what was done. But a history that helps you understand why those changes were necessary in the first place is rare. And if you're particularly unlucky, you'll find commits with titles like "oops", "fix of fix", "code review comments" and the like. And don't get me started on developers who mix unrelated changes with the real functional changes without being able to distinguish them.

* Another kind of traceability is what you collect from production environments in the form of logging. I often see two extremes here; too much logging where the trees can no longer be seen from the forest, or too little logging, which makes diagnosing production problems very difficult. And then you also have something like logging levels, where one warning puts you on the wrong foot, and the other error is not really that critical. And what about transition versus non-transient errors? Without good guidelines, don't expect developers to do logging the right way.

* With proper logging guidance you're not done yet. With the current trend of moving away from monolithical deployments, modern systems are composed of an increasingly number of (micro-)services. A simple user action can result in a message flow that runs through several of those services. In such a situation, it is almost impossible to follow what exactly happened, in what order, and by whom it was caused. In other words, correlation and causation are hard to determine.

Traceability is perhaps the most underexposed aspect of our profession, and if you ask me, organizations pay way too little attention to this. The developers do not find this important, do not feel like writing documentation, do not see the usefulness of a clear source control history, or simply do not know how to approach this and with what tools.

So what about you? Is traceability engrained in the engineering mentality of the people you work with? I would love to hear your thoughts. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions
