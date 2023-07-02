---

title: "Confluence, a wiki that will make people collaborate"
excerpt: ""

tags:
- collaboration
- documentation
- tooling
- team work

---

## Why do you care?

A very common discussion within organizations that do software development is what tool to use for documentation. Developers are usually pretty opinionated about that, if only because they want to ensure nothing gets in the way of doing the thing they love most: coding. Other people just want to be able to write down their notes _and_ being able to find them back. Although I love coding, I also really care about being able to track my notes and my breakdowns, but more importantly, to capture architectural decisions, development guidelines, and share technical information through internal blogs. 

But this seems to be a hard problem to solve. For example, do you recognize some of the below symptoms?

* Documents can be found in Microsoft Word, PowerPoint, [Miro](https://miro.com/), [SharePoint wikis](https://support.microsoft.com/en-us/office/create-and-edit-a-wiki-dc64f9c2-d1a2-44b5-ac59-b9d535551a32) and [Azure DevOps wikis](https://learn.microsoft.com/en-us/azure/devops/project/wiki/wiki-create-repo?view=azure-devops&tabs=browser)
* People use PowerPoint to capture notes during meetings.
* You'll find Word and PowerPoint documents all over the place, including OneDrive, folders within Microsoft Teams, local machines and SharePoint.
* Unless somebody remembers where a document is, there's no central place to find stuff.
* People are sending around copies of those documents over e-mail.
* Miro is used for structural documentation, because it's the most versatile tool they have.
* Everybody is continuously asking themselves were to put their meeting notes, brainstorm content and decisions.
* Developers love to use Markdown, some proprietary markup or Mermaid to write content and create diagrams, whereas non-developers just want to see what they are doing. 

For me, having a documentation platform that promotes collaboration is a crucial element of successful software architecture. Here are some of my main requirements.

* Being able to capture decisions, general documentation, technical breakdowns, temporary notes, all without being forced to use some kind of template. 
* Easy to use for people that prefer WYSIWYG editing (like me), but powerful enough for those that prefer to use Markdown syntax here and there (like me).
* The ability to publish internal blog posts to keep each other apprised of important technical and non-technical opportunities, or just to share your favorite tips and tricks.
* The ability to create diagrams without leaving the confines of the documentation platform. 
* It would be great to be able to create some kind of time line or roadmap to visualize proposed plans 
* Searching across all content should be a great experience with auto-completion that takes into account recent changes.
* To keep any RSI problems from reappearing, keyboard support is crucial for me. I prefer to use the mouse as little as possible. 

## Features drill-down
So given those requirements, I've been comparing SharePoint, Azure DevOps wikis (which are mostly the same as Github wikis) and [Atlassian Confluence](https://www.atlassian.com/software/confluence/features).

### Editing
Each product takes a different approach. SharePoint is a portal product, so editing is always _What You See Is What You Get_ (WYSIWYG). Azure DevOps (AZDO) and GitHub wikis are pure [Markdown editors](https://www.markdownguide.org/). Confluence is primarily a WYSIWYG editor, supports some parts of Markdown but provides almost every other option it supports using the `\` key. All products have some kind of versioning, but Confluence is the only one that allows you to compare multiple revisions in one go. 

Another big difference is how you start with a new page. SharePoint does support some kind of wiki feature, but that's a legacy feature that hasn't seen any development in over a decade. But everything else is always based on a strict template that is quite noisy and leaves little room for the actual content. Confluence always starts out with an empty page, but offers you about 130 templates that you can apply. Also, you can take any page and turn it into a template.

A nice feature that I value a lot myself is the personal space that Confluence offers. I often start creating some notes there without thinking too much about structure. Sometimes I leave it like that, but more often than not, I move that personal page into one of the existing spaces when it reaches some kind of quality level. There are no limits on how you can move pages around or reshuffle them in hierarchies. This also highlights another difference with the other products. In Confluence, everything is open for everyone, unless you choose otherwise. Most organizations that use SharePoint and Azure DevOps tend to lock everything down, which doesn't help collaboration. 

In Confluence, every space has an optional internal blog. This is mostly a special way to organize pages by year and month and any page can be converted to a blog post. AZDO doesn't have anything like that, but SharePoint has news posts, something that looks a little bit like that. Blog posts are a great way to share information with your colleagues without the need to keep them up-to-date. 

In terms of working with images, there aren't many differences. Both SharePoint and Confluence are very flexible. And you can paste images into AZDO, but changing its size, location and such requires Markdown syntax (or some custom CSS styling). Both SharePoint as well as Confluence support header images. A nice extra gimmick of Confluence is the status icon.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence1.png" class="align-center" style="max-width: 500px"/> 


## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


