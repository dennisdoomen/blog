---

title: 15 flags that tell you that your testability practices need maturing
excerpt: "In this fourth post of the series, I'll share some of the (red) flags I might raise while measuring the maturity of testing."

tags:
- architecture
- test driven development
- testability
---

With almost 25 years of experience, as a consultant, I help organizations in the .NET space to professionalize their entire software development efforts, from idea to production. During such visits, I get to scrutinize their development practices, quality standards, design principles, the tools they use, their deployment pipeline, the team dynamics, the requirements process and much more. In this series of short posts, I'll share some of the most common pain points I run into. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2021/symptoms-testability.jpg" class="align-center"/> 

In the [previous two posts]({% post_url 2021/2021-09-02-signs-architecture %}), we've been looking at the effect certain tools, techniques and architectural choices can have on developer productivity. But we never talked about how testability can influence that same productivity. No doubt every organization has this inheritent desire to make new functionality available to its clients as fast as possible. But many of them run into a (lack of) testability that prevents them to put that code in production with confidence. Here's a bunch of (red) flags I may raise when measuring the maturity of testing in the organization's mindset.

* Hiring professionals that have the exceptional skills to try to "brake" an application as well train the development team in improving the testability of a code-base, but then keep calling them testers instead of test engineers, engineers-in-test or QA engineers. 

* Not making a distinction between structured and unstructured testing and then wasting those unique skills and mindset I just mentioned on manual testing of test scenarios that could have been covered by fully automatic integration or UI tests. 

* Test engineers who are not properly involved in the grooming of functional stories resulting in little or no discussions on testability or the specific test cases that should have become part of the work breakdown. I've experienced myself how a good QA engineer managed to surface edge cases around usability, responsiveness and performance that the developers never could. 

* Manually testing a web application on multiple browsers or not properly using automated tools such as snapshot testing to detect unexpected visual differences between releases . 

* Unstable automated browser tests, or tests that seem to be stable for a while but then end up becoming unstable again. Even if you use the shiniest UI automation tool, you can't get away without building a decent test automation framework. And even then, brittleness is inevitable.

* Performance tests that are too far from reality to be really useful. Thinking times of users are not simulated well enough. The network traffic between the browser and the server doesn't take into account the real-life behavior of the JavaScript code. The test data in the database doesn't have a realistic distribution and size. Or browser caching causing too optimistic results because the test uses only a single user. 

* Insufficient understanding of what "concurrent users" really means or not getting the the distinction between terms like average response time, 90th percentiles and Apdex scores. Because of that, misinterpreting the results of a performance, load or stress tests happens more often than you might think. 

* Not using microbenchmarks to continuously measure the performance of critical modules or subsystems as part of the continuous integration pipeline in addition to regular performance tests.

* Code changes to web applications that can only be tested from the developer's machine, simply because the team does not have a mature deployment pipeline that allows any pull request to be deployed in a (temporary) cloud environment.  

* Automated tests that are not run as part of the pull request builds. Sometimes this is caused by a lack of a mature build pipeline, or by tests that just run too long. 

* Developers who write their unit tests after their production code, or don't do at all. Or those that write some code and then throw the result "over the wall" without even spending five minutes of manual testing. 

* Not using automated tests to verify the interaction with the database "because it's too difficult with our shared database server", resulting in bugs that won't surface until the system is put into production. Or not covering the markup code in React TSX/JSX files "because there's no business logic in it". 

* A test coverage level that’s way below the generally accepted norm of 90 percent, or the opposite, using that 90 percent as some kind of ultimate goal without using common sense. 

* (Dogmatically) choosing a single level of automated testing, i.e. developers that follow the test-per-class dogma making it very hard to refactor code. But equally wrong is the practice of only testing on component or module level, thus increasing the difficulty to understand the cause of failing tests and to properly cover all the test paths within that level. 

* Automated tests that need to be rewritten whenever the production code is being refactored. Often this is caused by architectural choices or automated tests that are too low-level. 

In retrospect, there's a lot to gain from well-designed and effective automated testing and there are a lot of things you can do wrong. But it remains a crucial ingredient of the recipe for a fully automated deployment, the ultimate goal for many software developers. 

So what do you think? Are you or your teams struggling with test automation as well? Any pain points I forgot to mention? I would love to hear your thoughts. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions
