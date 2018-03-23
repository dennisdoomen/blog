---
title: 'The Red Hot Developer - A model for removing distractions from your teams'
tags:
- team
- development
- agile
- collaboration
---

When your project, product or component gets sufficiently big that it has a large impact on the rest of the organization, you'll automatically get faced with lots of internal and external distractions. Other teams might want to get that pull request merged as soon as possible, all kinds of questions from sales and the project delivery people are piling up in Slack, and that occasionally failing UI automation build is also asking for some attention. And what about those weird errors in the log files and compile-time warnings that everybody keeps ignoring. Sure, we have a healthy amount of capacity reserved for technical improvements and reducing technical debt, but they tend to be used for the bigger things. 

![](/assets/images/posts/2018-03-23-pepper.jpg)

During [QCon New York](https://continuousimprover.com/tags/#qcon), I got introduced to the concept of the **Red Hot Developer** (or RHD for short), a (preferably) daily rotating dedicated developer that keeps distractions away from the rest of the team. He or she will investigate any failing builds and find the person who caused it, and try to find the answers for the questions those project people keep asking. Since the RHD is fully allocated to that task for the entire day, he (or she) may pick up some of those smaller left-overs. A typical day of being the RHD looks a little bit like this:

* Consult with the RHD from the previous day to see if there's anything to hand over.
* Monitor the builds. We use a couple of build lights spread around the floor that are controlled by [Beacon](https://continuousimprover.com/2015/02/a-beacon-of-light-in-shadow-of-failing.html), a little open-source project that connects TeamCity to a DelCom LED-powered light. So monitoring the builds is a pretty trivial job. 
* If there are any problems, either with the individual builds, the build infrastructure or anything else, the RHD will first try to investigate the problem. If it can be traced to a particular commit or developer, he'll notify that person and re-assign the problem. 
* Fix unstable unit or smoke tests, or find somebody to fix this. 
* Verify that all hotfixes on support branches have been merged to later branches.
* Serve as the first contact for people outside the team.
* Check that the performance metrics of the code hasn't decreased by observing the relevant builds.
* See if any questions from other teams or departments on Slack have been answered.
* Making sure pull requests from other teams are reviewed and handled correctly.
* Maintain frequently asked questions or refine the repository readme's.
* Work on small improvements to the build process, documentation or anything related to working with the code base and making sure existing problems are resolved properly.

When I initially proposed to introduce this principle, a lot of people objected against it. They didn’t like the idea of having to spend a full day on those petty tasks instead of working on the cool stuff they love to work on. Heck, some developers still don't like it. And the product owners weren't too happy about this either. They felt we were wasting valuable developers on stuff that didn't give them immediate business value. However, they didn't realize how much distraction the developers were suffering from when the entire collective was trying to do that same job. I still remember having to walk around the teams trying to find somebody to investigate a broken build or repair an unstable smoke test. 

As always when you introduce new ways of working, the RHD principle didn't have a smooth start. Some developers tried to keep working on their regular tasks. But after we empowered them to really block that day for RHD duties, the quality and stability of the code base started to improve considerably. That day really gave them the freedom and time to work on substantial things that gave a productivity boost to all developers. For instance, some of those seemingly smaller improvements ended up being a bit more work. Trying to transfer that work to the next person proved to be very inefficient. So at some point we allowed the RHD to continue the work the next day or to consider to pair up with the next RHD to ease the transition.  

A nice side-effect of being the RHD is that every developer gets to see almost every aspect of building, testing and deploying our code base. So front-end developers suddenly had to deal with RavenDB quirks or figure out how to configure WCF to use the right security context. Similarly, hard-core domain driven designers learned how to use Gulp and Webpack to improve the JavaScript and CSS bundles. This has definitely improved the flow of knowledge in the teams. And the business also learned to deal with this. At some point, they even checked the RHD schedule on Confluence if they wanted to get something done without going through our usual Slack channels. 

So what do you think? Do you recognize the cross-cutting concerns of being a developer? If so, would you consider trying the Red Hot Developer principle? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
