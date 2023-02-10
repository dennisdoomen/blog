---

title: "An opinionated definition of \"maintainable code\""
excerpt: "I've been looking for a better way to explain the difference between a developer complaining about quality and a product owner or manager."

tags:
- maintainability
- maturity
- codingpractices

---

One of the aspects of software development that I care deeply about is to ensure developers write maintainable code. Nothing is more a productivity killer than dealing with code written by a developer which only goal was to make it work. In other words, unless it's an Minimum Viable Product, you need to build the right thing the right way at the right time. That means that you don't only build what the business wants, but you also make sure it can be maintained. The "right time" is an additional factor that I added myself to emphasize that you need to be careful not to make things too complex. Maintainability is not an excuse for unnecessary abstractions or to "prepare the code for future extensions". Just keep it simple, stupid. You ain't gonna need that complexity until you need it. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/humorquote.jpg" class="align-center"/> 

But then what is maintainable code? Before I share my own definition, I have collected a bunch of quotes from the community, so let's start with those first. 

> Code that can be changed by any developer in the team without fear and with confidence.

I guess that says it all. Feeling confident enough to understand some code, modify it and being able to verify that you didn't break anything is one of the most important aspect of software development. Here's another one, this time by Cory House.

> Code is like humor. If you have to explain it, it's bad.

As somebody who often has to explain his jokes (and probably isn't very funny), I can totally relate to that. Of course this is a bit exaggerated, but the sentiment remains the same. The combination of the code itself, the naming used and any inline or function-level documentation should be enough to understand its purpose and how it accomplishes that, all without further explanation. 

To wrap it up, here's a quote by Michael Feathers, the author of  "Working Effectively with Legacy Code".

> Clean code always looks like it was written by someone who cares.

This is totally true. At the projects I work, I can often recognize the developer who wrote it, both in a positive as well as negative way. I've seen some particularly bad coding practices that I immediately recognize as coming from a particular developer, only to be confirmed by a quick "git blame". 

Now, for us developers, those quotes make total sense, right? But for a manager or product owner these are much more fuzzy or don't make sense at all. So for many years, I've been looking for a better way to explain the difference between a developer complaining about quality and a product owner or manager. It turns out they are actually talking about different kinds of quality. 

I'd like to distinguish those by using the terms "internal" and "external" quality. External quality deals with things users are facing. Things like bugs, usability issues, responsiveness and general performance issues. If that kind of quality is bad, you don't have to wait long before a product owner or manager will be ringing at your door. But when the internal quality is bad, nobody but the developers will be complaining, at least, for a while. Until your manager starts to notice that those features take more and more time to complete. 

But I suspect that _internal quality_ is still a bit vague for many, so I came up with the following definition:

* Maintainable code should be **readable** so it's clear what it tries to do and how it does that. This translates into good functional naming conventions, code that is well-factored and well-organized, thar clearly exposes its algorithm, and doesn't trigger a mental overload because it's too long or crams too much logic in as little lines as possible. 

* Maintainable code should be **testable** so that you can ensure yourself that the code-under-test is doing what it is supposed to. This is not as simple as having a bunch of automated tests. It means that you've thought hard about the boundaries within your system and how they align with the testing scope. Or better, you've carefully designed those boundaries by practicing a proven design principle like Test Driven Development. 

* Maintainable code applies **isolation** at the right places. In an ideal codebase you can change something in one part of the system without affecting the other parts. But that doesn't mean you need to go nuts with abstractions and Design Patterns. That can quickly lead to unnecessary complexity that you simple don't need. Finding the balance in where abstractions help and where they hurt is going to be the hardest part of your job (with the exception of working with other developers 😅). Not enough decoupling leads to spaghetti code. But too much decoupling leads to mental overload.

* Maintainable code has a high level of **traceability**. Not only do you want to know what the code intends to do, how it does that, and whether it actually does that. You also want to know why that code is doing that. This can be accomplished by something as simple as a little bit of extra documentation on the type or function. But it's much better if you have the full context. A clean source control history consisting of focused commits/changesets and a well-described pull request can make a world of difference. For the bigger picture, an architecture decision log or an internal blog is another way to get that understanding. 

* Maintainable code is also very **discoverable**. It is structured in a way that makes the underlying architecture apparent by using folder names that align with the internal decomposition. But the same applies if you look at the high-level architecture. If you make architecture posters readily available, it should help the more inexperienced developers understand which parts of that architecture are implemented by what part of the codebase. To summarize, discoverability makes it easier to understand where things belong and what dependencies should or should not exist. 

Of course I could have mentioned all the "ilities" we all learned during our education. And I haven't even mentioned engineering principles like Test Driven Development, SOLID, KISS, YAGNI and what not all. They are important, but for me, the five aspects of software development I mentioned today are the real foundation for writing testable and maintainable code. 

So what do you think about my definition? Does this resonate with you? Did I miss anything? Or do you completely disagree? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
