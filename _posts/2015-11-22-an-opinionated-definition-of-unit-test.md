---
title: "An opinionated definition of a unit test"
date: '2015-11-22T16:03:00.001+01:00'

tags: 
- unit tests
- integration tests
- test driven development
modified_time: '2015-11-22T18:56:15.145+01:00'
blogger_id: tag:blogger.com,1999:blog-15137028.post-5129091073366296361
blogger_orig_url: http://www.continuousimprover.com/2015/11/an-opinionated-definition-of-unit-test.html
---

During the same C# code reviews that triggered last week's blog post about writing 
[great unit tests]({{ site.baseurl }}{% post_url 2015-11-11-12-tips-to-write-unit-tests-that-dont %}), another discussion tends to pop-up, in particularly with new joiners (both experienced and junior): 

> "What's the difference between a unit test and an integration test?"

Well, in my opinion (which is heavily influenced by [The Art of Unit Testing](http://artofunittesting.com/"), a unit test has the following characteristics:  
* Fast (as in a couple of milliseconds)  
* Runs in-memory
* Has no dependencies on external I/O-bound resources such as networks, configuration files, databases and such  
* Can be run an infinite number times without failing
* Can be run in any order with the other tests  
* Doesn't cause any side-effects that might affect other tests or require manual clean-up  
* Cover one or more closely related classes

If a unit test does not meet ALL of these characteristics, it's an integration test. 

But what about subcutaneous tests, automated tests that cover everything from just below the user interface layer (which is usually technologically specific) all the way to the database? The goal of such a test is to cover as much as possible without relying on front-end technologies. So the question if it's a unit test is kind of irrelevant. It's more useful to discuss whether or not to include them in a CI build of some sort. And that all depends on how much they meet the earlier mentioned criteria. 

And what about tests that employ temporary in-memory databases such as Sqlite or RavenDB? Some would argue that the mere fact they rely on a database disqualifies them as a unit test. However, in our current project, we have hundreds of these tests. They run fast, cause no side-effects and don't have any dependencies. So yes, I usually treat these as unit tests as well. 

Another example in the grey area of unit tests is a test that uses OWIN to test the entire pipeline from the HTTP API all the way to the database level. In most cases, we'll use the same approach as subcutaneous tests and use an embedded Sqlite or RavenDB database. Technically they meet most of the characteristics of a unit test and we always run them as part of the CI build. However, whether I call them unit or integration tests depends highly on the scope of the test. If the purpose is to just test an HTTP API without involving the other layers, I tend to call them unit tests. Which brings me to the topic of the 'unit' in unit test.

Of all the things you need to think of while writing automated tests or practicing Test Driven Development, defining the scope of the unit you're testing is the hardest one. I still think Jeremy D. Miller's [Third Law of TDD](http://codebetter.com/jeremymiller/2006/05/30/achieve-better-results-by-following-jeremys-third-law-of-tdd-test-small-before-testing-big), in which he advises to test small before you test big, make sense. But small doesn’t mean a single class though. Instead, I recommend you to evaluate the collaboration of a couple of closely related classes and consider which of these change together when requirements change. Those might be a good candidate for the unit in a unit test. At the same time, having a couple of end-to-end integration tests is just as useful. Just don't follow any naive advice in which you need to choose between one or the other. Focusing on unit tests only might bring you in a situation were it’s difficult to refactor or redesign the internals of your code-base without breaking your test. Similarly, focusing on integration tests only will most definitely bring you into the debugger hell. Always choose the right tool for the right issue.

So what do you think? Does this definition make sense? If not, I'm very interested in hearing your thoughts. Let me know by participating in the Disqussions below. Oh and follow me at <a href="https://twitter.com/ddoomen">@ddoomen</a> to get regular updates on my everlasting quest for better solutions.</p>  