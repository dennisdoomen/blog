---

title: 8 signals that you don’t control your technical complexity
excerpt: "In this first post of a series, I'd like to focus on the challenges companies are facing while controlling their technical complexity."

tags:
- architecture
- complexity
- technical
---

With almost 25 years of experience, as a consultant, I help organizations in the .NET space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I’ll share some of the most common pain points I run into.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2021/8_signals_complexity.png" class="align-center"/> 

And I’m sorry to say that there’s plenty of those around, even if you don’t look too closely. But for this first post, I’d like to focus on the challenges companies are facing while controlling their technical complexity. Not only can that cause unnecessary delays in building new functionality, over-engineering and unwarranted complexity to jeopardizes maintainability, but it also makes it harder to find the experienced developers that can handle that kind of complexity. Here’s a couple of examples of what I ran into.

1. An architecture that doesn’t match the functional and technical requirements. Sometimes this happens because it was inspired by some technical hype the architect read about or picked up from visiting a popular software development conference. But I’ve also seen situations where a board of architects dictates a reference architecture that all systems must comply to.
1. Introducing technology because “it’s cool”, because it helps attracting new personnel, or simply because management bought into the golden mountains their developers, software vendors or architects promised them.
1. Exposing HTTP APIs or technical extension points to other teams or partners without considering the responsibilities that such decisions require, including well-defined versioning, backwards and forwards compatibility, proper developer documentation, and other necessities to guarantee the freedom to evolve them in the future.
1. Teams that add unnecessary complexity such as too many abstractions and extra layers that the current requirements do not warrant. Often this happens under the pretense of “being prepared for the future” or to make the system “better testable” (read as: overuse of mocking).
1. Architects and technical leads that love to boast with architectural principles, design patterns and generic heuristics needed to “build better software”, but end up pushing their dogmatic beliefs. Not because these principles are bad, but simplify because they are not experienced enough to understand the context in which those principles apply, as well as their pros and cons.
1. An imbalance between tactical and strategic architecture, with short-term solutions on one end of the scale and extremely ambitious (and unrealistic) castles in the sky on the other.
1. Teams that are struggling to understand and map the dependencies between the many components, building blocks, services and products, resulting in complicated plans that deal with cross-team initiatives.
1. And last but not least, developers that fail to find a way to work together on the same chunk of functionality, either because they don’t know how to swarm, pair or otherwise break down the work in tasks that allow working in relative isolation.

So what do you think? Do you recognize any of these pain points? Any additional challenges that you run into while trying to keep your complexity in check? I would love to hear your thoughts by commenting below. 

Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.

