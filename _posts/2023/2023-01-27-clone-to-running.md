---

title: "From clone to running a code base in under 5 minutes"
excerpt: "If I can't clone your repository and get the application running in under 5 minutes, you're doing something wrong"

tags:
- legacy
- maintenance
- building
---

Whenever I have to deal with an existing codebase for the first time, it always reminds me of the following quote:  
> If I can't clone your repository and get the application running in under 5 minutes, you're doing something wrong

I don't remember exactly who said it, and I couldn't find the original source on Google, but there's truth in that statement. Don't get me wrong though. Just like a code coverage of at least 90%, this 5 minutes should never be a goal. But it's another nice metric to see how mature you or your teams are. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/desperate.jpg" class="align-center"/> 

Like everything in our profession, it's not an exact science. But imagine you're getting the source code of a some application to build and run it. You might be asking yourself a lot of questions:

* Do you I to setup NPM and NuGet credentials myself? And if so, how?
* What tools do I need? A specific .NET SDK? a specific Node version? Do I need NPM or Yarn?
* How do I build the backend and the frontend? Is the order important?
* Do I need to setup users and roles to use the application? If so, how?
* How do I setup the database schema? Can it work with SQL Server LocalDB or do I need something else?
* Can the application run without master data? If not, is this imported automatically from somewhere?
* What other dependencies does this application have? Do I need to install them myself? And how do I connect them together?
* Do I need to run a release or a debug build? And can I change that?

And that’s only for running the application. Just thinking about all of that surely takes more than 5 minutes already. And what about trying to understand the architecture and the development process? You'll have even more questions.

* Are there any introduction labs?
* Where can I find some high level docs?
* What kind of release strategy is the repo following? GitFlow? GithubFlow? Something else? 
* How is the application versioned? Do versions have some kind of semantic value? 
* Where are the builds? And which build is used for what branch?
* Where are artifacts such as NuGet or NPM packages pushed?
* Who maintains this repository? 
* How can I contribute? What are the rules for pull requests (if any)? 
* What does the folder structure look like? Can I deduce some kind of architecture style from that?

The solution to all of this is pretty simple. The first list of questions can be covered by having a nice and clean build script that takes care of everything that you need to get the code to build and run. It may prompt you for the credentials to your NPM/NuGet feeds. It will download the necessary tools or libraries automatically or tell you exactly what you need to install. It may even provide you with a little command-line menu where you can choose the necessary steps to get the application up-and-running. 

Does that mean you never have to understand what's happening under the hood? Of course not. But it should get you started without being overwhelmed by too much details. That's why the second set of questions needs to be covered by a concise but complete and well-structured read-me, potentially amended with additional guidance on how to contribute in the most efficient way. Especially if you're on Github, this should be an effortless exercise. 

So what about your project? Did you take the necessary steps to make this onboarding process as simple as possible? If so, are you sure? When was the last time somebody had to get to know your code base? Let me know by commenting below. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen) and [Mastadon](https://mastodon.social/@ddoomen).
