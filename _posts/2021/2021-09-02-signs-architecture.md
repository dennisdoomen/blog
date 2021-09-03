---

title: 8 signs that your architecture is causing your developers to stumble over each other
excerpt: "In this third post of the series, I'll share my perspective on how architectural constraints and decisions can hamper productivity."

tags:
- productivity
- architecture
- software development
---

With almost 25 years of experience, as a consultant, I help organizations in the .NET space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I'll share some of the most common pain points I run into. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2021/8_signs_architecture.jpg" class="align-center"/> 

In my [last post](/2021/07/opportunities_productivity), I talked about how the (mis)use of tools and techniques can affect the productivity of software developers. Today, I want to talk about productivity again, but then from the perspective of the constraints imposed by architecture and technical design choices. Unfortunately, unless you're a very experienced software architect, these kinds of constraints are much more difficult to spot. Often they come from dogmatic use of certain principles, practices or libraries, or simply because of a lack of experience. I've seen this firsthand more than once, so let me share some examples that should make you frown.

* A monolithic system has quite a negative connotation for many people. But for me, it shouldn't mean much more than that it is rolled out as a single unit, and surely doesn't imply a big ball of mud or a spaghetti code base. A monolith can very well be designed using nicely isolated internal modules with well-controlled dependencies. Unfortunately, that's not what I see in reality. Without those well-defined internal boundaries, teams will be stepping on each other's toes all day long hampering productivity on all levels. Even worse, such a system is often used as the main argument for fully jumping on the micro-services train. 

* Organizations that struggle to scale their software development efforts often blame this on the architecture of the system, such as the monolith we just mentioned. But what's worse is that this argument is then used to create a case for rebuilding the system, something that developers that love building new stuff will eagerly support. I'm still convinced that in the majority of cases, rebuilding a system that has been in production successfully for many years is a wasteful investment. There's plenty of techniques to gradually improve existing systems, provided you're committed to set-up a good team structure for that. 

* Bugs appearing in places that seems to be completely unrelated to the code that was changed in the first place. Sometimes those dependencies can be traced down pretty quickly, but often the root cause is much more subtle and caused by (global) variables being shared across threads or module boundaries, or injected through a dependency injection framework. I have seen developers spending days trying to find the real issue.  

* A system with a layered architecture sounds really nice on paper, with clear separation between the business logic that is so core to the domain and the more technical aspects such as the data access layer.  But in reality, this is quite an obsolete architecture style resulting in too much generic code, for which it's often unclear how it contributes to the business problem at hand. Optimizing generic code is always hard, since you can't really predict the use cases. And that's not all. Often, a layered architecture results in too much coupling, making it hard to understand the consequences of changing the generic code. 

* By now, most senior developers know SOLID - a set of design principles that help make code better understandable and adaptable - quite well. But just like every principle or pattern, you can be overzealous. Too many layers and abstractions, caused by dogmatically applying SOLID will make it much harder to understand and relate the various parts of your code base. Finding the right balance isn't easy however, so a lot of developers get it wrong. 

* A typical trait most software developers have is that they don't want to repeat themselves. They don't need a lot of encouragement to rewrite code that seems to be duplicated within the same codebase into some reusable, preferably in a common project. A recurring argument for this is that by doing that, you won't have to fix a bug twice. This principle, that is so eloquently named Don't Repeat Yourself, or DRY in short, is one of the engineering principles that aspiring developers are being taught by their more senior colleagues. Although DRY is generally a good principle, it can also lead to too much generic code, and thus, too much coupling. And we already know by now that this is a serious productivity killer. 

* Somebody wiser than me once said that if it takes more than five minutes for a new developer to download your code and run the application, you're doing something wrong. One reason I'm seeing often is when an organization has become too attached to a specific product for building your software. A good example of this is when an organization has adopted Microsoft's Azure DevOps build pipelines, making it very hard to build locally. That you put your code in an online Git repository is a great thing to do. But for the rest, try to allow your developers to work as independent as possible from online services. 

* Another way to measure the independence between teams is to see how difficult it is for another team to contribute code changes to a code base they don't work on, and even deploy it somewhere for testing. It's sometimes astonishing how many teams get blocked by planning ceremonies, strictly controlled access to code and other impeding enterprisy governance. 

To wrap up, there's plenty of things developers do that seem to be very useful and according to the book, but which will turn out to be causing a lot of impediments, both in development productivity and quality levels. And just like I concluded in my last post, finding the right balance and the appropriate practices is the big challenge here.

So what do you think? Do you recognize any of these architectural challenges? Anything else holding your development teams back? I would love to hear your thoughts. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.