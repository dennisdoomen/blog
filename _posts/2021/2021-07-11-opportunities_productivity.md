---

title: 12 opportunities to improve the productivity of your development teams
excerpt: "In this second post of the series, I'll discuss the opportunities organizations miss to improve the productivity of their teams."

tags:
- productivity
- programming
- software development
- collaboration
- refactoring
---
With almost 25 years of experience, as a consultant, I help organizations in the Microsoft space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development 
practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I’ll share some of the most common pain points I run into.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2021/12_opportunities.png" class="align-center"/> 

In my [last post]({% post_url 2021/2021-06-22-signals-technical-complexity %}), I talked about keeping some grip on the complexity that some development teams like to introduce. This time, I want to discuss the opportunities some organizations miss to improve the productivity of their teams. Oftentimes, when you talk to the developers themselves, it’s quite evident what they need. But this never lands well at the management level. A simple 50 bucks productivity tool requested by a developer can save you thousands of dollar. But that’s just an example. Here are a couple of more.

1. Developers who don’t see refactoring as a central element in their day to day software development efforts, or managers that see this as a waste of time that just introduces more risk in delivering new functionality. They are often caused by lack of experience with professional software development, not using a proper IDE supporting refactoring, or working with dynamic languages such as JavaScript. The result of this is an abundance of “dead” code, weird or obsolete naming, unacceptable large chunks of code, or other so-called “code smells” that jeopardize the maintainable of your code base.
1. Technical debt that is not under control and that isn’t being payed back regularly. Sometimes it’s the manager that doesn’t get the benefits, sometimes it’s the pressure that team feels on delivering new functionality, and sometimes it’s the team itself who don’t get why it’s so important to get rid of this. But oftentimes, it’s a combination of all three.
1. Developers who’s only goal is to get the code to work as efficient as possible, and fail to think of the structure of that code to support maintainability. They gloat on principles like SOLID and refuse to refactor towards Design Patterns, resulting in more and more problems trying to introduce new functionality. However, that coin has a flipside as well, where tech leads put too much emphasis on design patterns and principles resulting in unnecessary rework and so many layers of abstractions that nobody understands the code anymore.
1. Organisaties that try to pursuit a certain architecture with well-defined boundaries and contracts, but fail to physically organize their teams to align the communication with those boundaries. Or, looking from a different perspective, organisaties that treat architecture, tools, people, processes and infrastructure as independent ingredients of software development.
1. Pull Requests (if used at all) with hundreds of files, often filled with unrelated changed and commits with titles where the lack of structure of the involved developer is clearly visible. Sometimes those changes include refactorings making it even harder to differentiate the functionally relevant changes from the other noise.
1. An obvious consequence of this is that peer reviews either don’t happen at all, very superficial, or may take days or even weeks. But I also see plenty of teams that skip the review altogether and directly merge changes into a central code base, without any automatic safety net. I mean, Continuous Deployment is a great goal, but it also requires a strong and mature organization to pull it off.
1. Teams where developers work in isolation, resulting in user stories that make insufficient progress, and where it’s too much to ask to deliver functionality within a single sprint. Often excuses are used such as “pair programming doesn’t work for us”, or “it’s impossible to break down this work so we can swarm on it”.
1. Repositories that don’t have an obvious release strategy and for which the release numbers don’t seem to have a specific meaning in terms of backwards compatibility, bug fixes or new functionally.
1. Developers are like ordinary people, and ordinary people have personal preferences to make them feel comfortable. So, it shouldn’t come as a surprise that developers love to the use the tools that they believe will make them efficient. However, the bigger the organization, the more there is a tendency to standardize, without realizing how much ineffective developers are costing them. Or worse, I’ve seen developers leave a company out of pure frustration because they are forced to work with tools they hate.
1. Developers that stick to either CLI tools on one end of the scale or tools with a graphical user interface on the other. I see those extremes with companies that are more in the Microsoft Azure DevOps corner, but also in those organizations that prefer open-source software. And this is strange, since combining the best of both worlds might make you much more efficient.
1. A nice example of this is the overzealous use of email for threaded discussions, using chat tools like Slack and Teams for structured communication, and the adoption of Confluence, Sharepoint and other wiki-like products for unstructured documentation. Every tool has its pros and cons. So trying to force a single tool for everything can be extremely costly at best.
1. The lack of innovation, for example by failing to reserve time for that. And it’s not only the developer teams that complain about that. It’s also management that often does not understand why their teams stopped being innovative. Sometimes, you’ll find a separate department tasked with innovation, even though innovation should be a continuous thing that comes from the development teams. An obvious smell of this is when developers don’t get the freedom to try out new tools or techniques.

So what do you think? Do you recognize any of these opportunities to improve productivity? Anything else holding your development teams back? I would love to hear your thoughts by commenting below.

Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.