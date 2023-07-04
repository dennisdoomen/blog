---

title: "Confluence, a wiki that will make people collaborate on documentation"
excerpt: "An in-depth evaluation of SharePoint, Azure DevOps Wikis and Atlassian Confluence as a documentation and collaboration platform"

tags:
- collaboration
- documentation
- tooling
- team work
- confluence
- azure-devops
- sharepoint
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

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence1.png" class="align-center" /> 

But in addition to the refined editing experience, Confluence offers two additional USPs. One is the ability to have two or more people edit the same page at the same time, just like you can with Office documents stored on SharePoint or OneDrive. This is a crucial feature to make collaboration during meetings possible. 

The other USP is the mobile app. You might wonder why you want to edit on a mobile app. But as a developer, I often come up with ideas or insights during breaks, workouts or while away. Being able to quickly add some notes through the mobile app really helped the creative process software development really is.

### Collaboration
There's no point in producing a page of documentation without making it easy for other colleagues to review and contribute. All products in this comparison support page-level comments, but only Confluence allows you to leave a comment on a specific line (or part of it) and have a follow-up discussion. Also, unlike Azure DevOps, both SharePoint and Confluence allow you to like a page, but Confluence allows you to use emojis as well. Tagging somebody in a page using e.g. `@dennisdoomen` only works on AZDO and Confluence. 

When somebody modifies a page you created or touched, Confluence will automatically make you a _follower_ and send you notifications. You can choose whether you want to get those notifications directly, at the end of the month, or once a week. I think you do the same in SharePoint, but I couldn't find any similar options in AZDO. In fact, unless you explicitly follow a page, you will not get notifications whatsoever (which is a shame). All products are sophisticated enough to allow you to take a link to a paragraph header, but only Confluence will copy that link to the clipboard just by clicking an icon next to the header. 

Unique Confluence features that I use every day are the ability to create actions, right in a page and assign those to people. Don't get me wrong. They are not meant to replace a proper work item tracking solution. I typically use them to identify follow-up questions or things to complete a particular page. Those tasks will show up next to your profile information, but you can also create a page to list all tasks for a particular space or group of pages. Similarly, when collecting notes in meetings, I often use the decision macro to track important agreements. But you can easily use another macro to generate a list of those decisions on another page.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence2.png" class="align-center" /> 

### Tables
As an architect, I use tables a lot. They help me structure my thoughts in a presentable way and are a perfect technique to compare multiple options when trying to determine the pros and cons. All three products in this evaluation support creating tables, but the difference is big. 

The least powerful is Azure DevOps, which relies on Markdown and the corresponding [cryptic method](https://www.markdownguide.org/extended-syntax/#tables) of using pipes to model columns and rows. Yes, there's a button to quickly create an initial table and you can align columns left and right if you know the syntax, but that's it. No colors. No merging of columns, no sorting. 

SharePoint has a more WYSIWYG experience and provides two options for creating tables. One is based on a primary concept in SharePoint called lists, but which is less suitable for wiki pages. The other one is part of the news/wiki functionality and allow you to create tables the way you expect to. It provides various options to align columns, add or remove rows, add images in cells and a couple of predefined coloring options. 

However, none of that is comparable to Confluence where you create a new table using the toolbar or using the `/table` command (which auto-completes). But it’s the UI experience that really shines in the way you can order, sort, assign colors to columns, cells and rows and where most actions require only a single click. You can even specify how that table should fill the horizontal space of that page. So it's totally fine to have the table span the entire page, even if the page itself is using the default width optimized for reading. Oh, and it's only product that support merging cells. 

But just as with everything else in Confluence, it's the little things that make it such a great product. If your table is big and doesn't entirely fit on the screen, it will provide proper scrolling capabilities _and_ keep the header column and rows always visible. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence3.png" class="align-center" /> 

But that's not all. You can also dynamically build tables from data on other pages. I use that a lot to build a decision log from a set of pages representing architectural design decisions. And what do you think of being able to visualize table data in graphs, just like you can in Excel?

### Visual content
In addition to creating graphs, there's a a lot more want to present on a page. Azure DevOps has the least amount of options, although it's the only one that supports Mermaid syntax and formulas without the need to install something from a Marketplace. Both Confluence and SharePoint support tons of options for displaying embedded Excel sheets, PowerPoint presentations, images, videos and such. Confluence adds to this embedded diagrams using [DrawIO](https://www.drawio.com/blog/embed-diagrams-confluence-server), something I use to add [C4](https://c4model.com/) and UML diagrams to technical documentation, and visual roadmaps. Both Azure DevOps and Confluence support generating a table of contents, but Confluence has a huge marketplace with free and paid plug-ins to visualize other kind of content. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence4.png" class="align-center" /> 

### Searching and navigation
All products provide reasonably decent search capabilities and all will try to load results as soon as you start typing. Although SharePoint only includes pages in the list of results when you start typing, it can search in Office documents. They won't show up until you ask it for more results though. Azure DevOps seems to have the most flaky behavior. Sometimes it doesn't show any results until you press ENTER, sometimes it just lists the number of items it found, and sometimes you get a list of prefixes you can use. It might be me, but I'm missing the logic behind that. It's also the only one that does _not_ show recent searches. Confluence is the only one that allows searching for pages based on a label or tag, and also is the only one that supports searching using partial keywords. So, unlike the other products, you _can_ find my content by searching for "enni". And one neat feature that both Confluence as well as SharePoint have are related pages. In Confluence, it even provides cues on how it determined that those are related. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence5.png" class="align-center" /> 

### Usability
From a usability standpoint there's not much in Azure DevOps and SharePoint that is worth mentioning. This is clearly not a priority for Microsoft. And that's the complete opposite for Confluence. From my experience, Atlassian really tries to understand how people are using the product and keeps making the writing and collaboration experience better. 

Practically every macro or construct is available using the `/` key and the options to enrich that original blank page are numerous. It also understands that long lines don't help with readability and optimizes the page width for that (which you can always override if you need to). And you never have to be afraid that you loose your changes when your internet connection goes down, your browser crashes or you accidentally closed a tab. Confluence will keep your (draft) notes around, both on-line and off-line. And did I mention the ability edit the same page at the same time, just like you can in Office? This works beautifully during meetings because you can see each other's cursor and can work on the content at the same time, even with multiple people. 

### Other features
Moving content in and out of the three products works quite differently. SharePoint is a portal product and is closely integrated with Office 365, so it's designed for displaying all kinds of content. Azure DevOps doesn't do any kind of integration, but Confluence allows you to upload Office files and edit them using a companion app you need to install locally. It's workable, but not comparable to the SharePoint or OneDrive editing experience. On the other hand, unlike Confluence, pasting links in Azure DevOps or SharePoint doesn't do more than creating clickable phrases of text. Compare that to how Confluence renders links:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence6.png" class="align-center" /> 

And pasting links to JIRA, GitHub or AZDO issues provides a similar experiences. When you do, it'll ask you to authenticate with the corresponding service (if needed) and then render them in a similar fashion.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/confluence7.png" class="align-center" /> 

Every product supports exporting a page to PDF using the PDF printer that comes with Windows (which, I assume, works the same on other platforms). Confluence adds native support for exporting to Word or PDF to that. However, if  you heavily rely on wide-screen tables and emojis, it doesn't look that great unfortunately. 

Copy and pasting between products is always a painful topic, especially if you're moving from one to the other. When you copy something to SharePoint, you'll often spend quite some time trying to get the formatting right. Using Windows' copy-as-text can help here. Copying anything to Azure DevOps wikis basically means copying some text and then manually adding all the Markdown to repair the links, fix the tables and such. Again, Confluence is a different beast. It's smart enough to convert headings to the correct heading type, and since Confluence only uses a single font type and size, you will never have to fiddle around with formatting issues. I particularly liked how you can copy a page from an Azure DevOps wiki to Confluence and keep all the headings, images and such. 

### A detailed comparison
This completes the deep-dive and comparison of the three products. But before getting to the conclusion, here's a table listing some of the important differences.

| | SharePoint | Azure DevOps Wikis | Confluence |
|-|------------| ------------------ | ---------- |
| Version control | ✅ | ✅ | ✅ |
| Keyboard shortcuts | ❌ | ✅ | ✅ |
| WYSIWYG editing | ✅ | ❌ | ✅ |
| Markdown support | ❌ | ✅ | ✅ |
| Integrated diagrams | ❌ | ✅ (through Mermaid) | ✅ (Draw.io) |
| Blog | ✅ | ✅ | ✅ |
| Personal pages | ❌ | ❌ | ✅ |
| Searching
| - Live completion | ✅ | ✅ | ✅ |
| - Recent searches | ✅ | ❌ | ✅ |
| - Partial keywords | ❌ | ❌ | ✅ | 
| - By label/tags | ❌ | ❌ | ✅ | 
| Tables
| - Create/modify | WYSIWYG | Markdown | WYSIWYG |
| - Sorting | ✅ | ❌ | ✅ |
| - Colors | ❌ | ❌ | ✅ | 
| - Column merging | ❌ | ❌ | ✅ | 
| - Keep headers visible while scrolling | ❌ | ❌ | ✅ | 
| Collaboration
| - In-line comments | ❌ | ❌ | ✅ | 
| - Page-level comments | ✅ | ✅ | ✅ |
| - Like pages | ✅ | ❌ | ✅ | 
| - Emojis | ✅ | ✅ | ✅ |
| - Tagging people  | ❌ | ✅ | ✅ |
| - Assign actions | ❌ | ❌ | ✅ |
| - Live editing | ❌ | ❌ | ✅ |
| Import/export content
| - Images | ✅ | ✅ | ✅ |
| - other content | ✅ | ❌ | ✅ |
| Access management 
| - Default | Closed | Open | Open |
| - Page-level permissions | ❌ | ❌ | ✅ |
| Notifications | ✅ | ✅ | ✅ |
| Follow entire wiki | ❌ | ❌ | ✅ |
| Android / iOS app | ❌ | ❌ | ✅ |
| Auto-saving (i.e. crashes) | ❌ | ❌ | ✅ |
| Templates | ✅ (forced) | ❌ | ✅ (opt-in) |
| Integration with AZDO/JIRA/Github | ❌ | ✅ | ✅ |
| Roadmapping | ❌ | ❌ | ✅ |
| Clickable headers | ❌ | ✅ | ✅ |
| Reorganize pages | ✅ (only within site) | ✅ (only within project) | ✅ (everywhere) |
| Edit Office documents | ✅ | ✅ | ✅ |
| Automatic lists (e.g. a decision log) | ✅ (using SharePoint Lists) | ❌ | ✅ |
| Wide-screen support | ❌ | ❌ | ✅ |
| Immersive reading | ✅ | ❌ | ✅ |
| Single Sign-on | ✅ | ✅ | ✅ |

### In conclusion
Even though a lot of people think these products are alike, I've essentially been comparing apples and oranges. SharePoint is a portal product which is designed to build visually appealing intranets, share news and provide lists of Office documents. Azure DevOps offers a non-WYSIWYG wiki, which in terms of functionality is nothing more than that: a wiki like we know from the past. The only real documentation and collaboration platform in this comparison is Confluence. I'm a big fan of many of the Microsoft products, but if you are talking about the best of breed, then Atlassian is miles ahead of the other products. In fact, it's not even a real comparison. Confluence offers such a refined experience, that I've seen first-hand how people stop using email and Office documents altogether. 

## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 26 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


