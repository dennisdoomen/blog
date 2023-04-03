---

title: "Are you at your Plateau of Productivity yet? "
excerpt: "A little story about the typical adoption process of TDD and how that works for other tools, principles and practices"

tags:
- solid
- test-driven-development
- best-practices
---

Here's a little story:
> Imagine you've been attending that conference where you first learned about that cool new thing, let's say, something like Test Driven Development. Your first reaction might have been "Meh, not for me". You were not seeing the value just yet and could already imagine yourself having to explain your manager why development of a new feature is taking so much time. But then, over time, the idea of TDD starts to sink in and you start to experiment with writing tests first. 
> 
> Week by week, you become more enthusiastic and really start to grasp how powerful TDD is. You start to tell your colleagues about it, offer to run some internal demos, and even decide to write a couple of blog posts about it. Since you're not afraid to talk in public, you manage to convince your boss that you doing a full-blown presentation is a great idea. And since it is, and you did a great job, you even got accepted at a local event to convince the community about how great TDD is. And hey, and since you're a millennial, you record a YouTube video of that presentation and decide to invest in building a real Pluralsight training on the many advantages of TDD. 
>
> In the meantime, you've been applying this test-first mindset quite rigorously and have discovered that it is much harder than you thought. You and your team wrote tons of small and focused unit tests and the number of regression decreased significantly. So even though your manager doesn't like the extra time it costs to build a new feature, he does like the improved quality. On the other hand, a lot of your colleagues start to complain about having to rewrite a lot of those tests every time they change some functionality. In fact, it gets so bad that you start to believe that in retrospect, adopting TDD wasn't such a great idea after all.
>
> As a consequence of this, you start to talk to other teams and recommend them not to even bother with TDD. You mention the time it'll cost you to write tests before writing code, and particularly complain about all the rewriting you have to do all the time. And as you've become part of the community, you write a blog post about it, build a presentation titled "The Dark Side of TDD" and record a popular YouTube video with the catchy title "10 reasons why shouldn't use TDD". From a proponent, you've completely switched to become an opponent. 
> 
> But then, many years later, you've finally discovered that TDD is really as great as you initially thought. In fact, you can't even write code without first writing some tests anymore. It's really like driving without seatbelts for you. You just needed to make sure you stay away from any dogmatic ideas. You now know a class is not the default scope for a unit tests and that TDD is equally applicable to JavaScript and TypeScript as it is to C# and Java. Also, you finally realized that understanding the internal seams of your system is crucial to be successful with TDD. But you've accepted that first trying to sketch out the overall design of your classes without writing a single test is totally fine. You know quite well that first defining the responsibilities, the ownership of data and how those classes will work together is how software development works. You've finally found the sweet spot.

Recognize anything in this not-so-made-up history lesson? There's a pattern in this that happens a lot when discovering some new technology, practice or principle. Gartner made this very visual through their Hype Cycle for Technology Adoption. It looks like this:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/gartner-hype-cycle.png" class="align-center"/> 

You can clearly see the points in time as they happened in my little story. The Peak of Inflated Expectations is when you became 100% convinced that TDD is "the way" and tried to convince everybody of the same "fact". The Trough of Disillusionment is when you discovered the flipside of TDD and started to tell everybody to stay away from it. The Slope of Enlightenment is that period of time where you learned the pros and cons, ditched any dogmatic believes and finally became a proficient practitioner of TDD. 

So the next time you become overly zealous about some new technology, principle or practice, ask yourself where you are at the technology hype cycle. Whether you're looking at the cloud as a solution for your scalability problems, microservices to move away from that monolith, Event Sourcing, Onion Architectures or the next JavaScript framework, keep those feet on the ground and challenge yourself. It puts things in perspective and helps you avoid going down the rabbit hole. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).