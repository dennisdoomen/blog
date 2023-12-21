---

title: "22 reasons to ditch Azure DevOps and switch to GitHub as soon as possible"
excerpt: "I resent every day I have to work with AZDO. Not because of personal feelings, but because I've seen myself how it is holding back the teams I work with to collaborate more efficiently and commit code that has a high level of traceability."

tags:
- collaboration
- tooling
- git
- github
- source control
- inner sourcing
---

As an open-source maintainer for over 15 years, and an [open-source project](https://fluentassertions.com/) with over [300 million downloads on NuGet](https://www.nuget.org/packages/FluentAssertions), I like to think I know what it takes to have large numbers of people contribute to a code-base efficiently. Next to that, I've been a consultant for almost 27 years helping organizations to get the most out of modern software development efforts. As such, I regularly work with Azure DevOps (AZDO), GitHub and even BitBucket and have been able to experience their differences first-hand. In this post, I'm going to give you 22 reasons why you should switch to GitHub for your source control as soon as possible. 

## Collaboration

1. I've [written](https://www.continuousimprover.com/2020/03/keep-source-control-history-clean.html#dealing-with-code-review-comments) about this before, but suffice to say I care a lot about a clean source control history. Having feature branches mixed up with shared branches like main, develop, hotfix/x.x isn't just noisy, it will often seriously obscure the visual graph of your commit history. The obvious solution to that is to use personal forks and create pull requests to bring your changes back into the main repository. AZDO _does_ support a form of forks, but they are really just additional repositories in the same project as the main repo. It clearly was an afterthought. Just imagine a project with 50 developers. Compare this to Github where forks are completely hidden unless you look for them. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-forks.png" class="align-center" /> 

1. I'm a big proponent of Inner Sourcing where teams contribute to eachother's repositories, just like people do in open-source projects. It's also the perfect model for platform teams where shared components and infrastructure is build for other teams. Being able to fork _any_ repository within the organization, fix a bug or add a feature, and submit a pull request for review without prior permission is crucial for for this. In AZDO, everything is locked down by default. Repositories are created under a project, and nobody has access to that project, unless access was granted. And not only that, because AZDO treats forks as projects, you must have permissions to create a repository in that project. You can work around all of this to a certain extend, but for me these are all demotivating factors for adopting Inner Sourcing.

## Pull Requests and Reviews

{:start="3"}
1. The first thing I noticed when I had to review a pull request in AZDO is how little space is left for the actual file diff. The below view shows AZDO with as much parts of the UI collapsed as possible.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/azdo-view.png" class="align-center" /> 

    The toolbar on the left and the entire top header section remains visible (including the Side-by-Side button), even if you scroll down. 

    Now compare this to the same file and same revision on GitHub. Notice that the entire top bar with your profile information and commit information is hidden so to keep as much screen real estate available for the diff. Also notice the readability of the changes lines compared to AZDO. For reference, both screenshots were made with the same browser and zooming set to 100%. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-diff.png" class="align-center" /> 

    Only when you scroll the view all the way to the top, then the rest of the information will reappear. GitHub is full with UX optimizations like that and keeps improving it. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-diff2.png" class="align-center" /> 

3. While implementing a feature or fixing a bug, I almost always run into refactoring opportunities and potential naming improvements. To avoid polluting the functional changes with those refactorings, I try to separate those changes into separate commits. The idea is that it'll make it easier for the reviewer to understand my changes, resulting in a quicker and more thorough review. Unfortunately, AZDO doesn't properly support that. You _can_ create a pull request with multiple commits, but the review comments on those commits will _not_ be visible on the resulting pull request. In GitHub you can review one or more commits at the same time and easily browse back and forth between them. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-commits.png" class="align-center" /> 

3. As a single pull request with multiple commits doesn't really work, I tried another approach where I push my changes to a branch, create a pull request, and then create a new branch from the previous one. By stacking those branches on top of each other, I can continue delivering my changes in a small chunks for easy reviewing and a clean history. But even that isn't properly supported in AZDO. You _can_ create a pull request from the branch that was stacked on top the previous one, but as soon as the original pull request is merged, AZDO gets confused. It'll show the correct commits between the feature branch and the target branch of the pull request in the Commits tab, but the Files tab keeps showing files from the previous pull request this branch was based off. The only workaround is to recreate the pull request from scratch. But that is a pain if you tend to add an extensive rationale to every pull request. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/azdo-files-tab.png" class="align-center" /> 

3. A code review is a serious process that requires the reviewer to thoroughly understand the context of the changes and the changes themselves and provide well-written and thoughtfull comments. For example, I often use Emojis to emphasize my intention and help myself from bitpicking too much. GitHub allows you to submit individual comments, just like in AZDO, but it's much more common to first complete the review and _then_ submit them as one batch. Because of this, GitHub understands which comments belong together and will visually group them so to keep the comments from different reviewers organized. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-comment-panel.png" class="align-center" /> 

    A nice extra is that you can revise a previous comment before you submit the entire review. In fact, you can edit multiple comment at the same time.Sometimes I realize something while adding a code review comment and want to quickly update a previous comment before finalizing the current. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-resolved-comments.png" class="align-center" /> 

3. AZDO doesn't allow me to make a distinction between review comments that need to be resolved or general comments on the pull request. So if I want to ping somebody so they are aware of the pull request or leave any other kind of comment like a link to a related pull request or issue, that comment will need to be "resolved" to unblock the pull request. In GitHub, you can make that decision per comment.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-start-review.png" class="align-center" /> 

3. As I often commit my changes into a temporary commit using `git save`, I also keep amending and force-pushing the follow-up changes to that same commit. I do the same when processing code review comments and pushing [fix-up commits](https://www.continuousimprover.com/2020/03/keep-source-control-history-clean.html ). In AZDO, there's no way to see what changes that force push overwrote. Compare this to GitHub, which adds a nice clickable link to see the differences. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-force-push-diff.png" class="align-center" /> 

3. Both AZDO and GitHub will hide the unchanged lines in the diff viewer. This is nice and makes it easier to review the relevant changes. But sometimes, you also want to see the context of the change. GitHub allows you to incrementally expand the diff to show the lines above or below. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-expand-diff.png" class="align-center" /> 

3. In AZDO, creating a link to a specific line in a file that is part of a pull request is cumbersome. Also, there's no way to create a link to multiple lines of text in such a PR. I often use that to refer to similar changes from a comment. In GitHub, you can SHIFT-click on a set of a lines to get a friendly URL you can share.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-block-link.png" class="align-center" /> 

3. In GitHub, you can directly edit a file that is a part of a pull request and which source branch might be even on a different fork. It'll open a new window to edit the file and push a new commit directly to the source branch.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-edit-file.png" class="align-center" /> 

    And if you don't have write access, GitHub will ask you whether you want to create a branch instead and use a pull request to the contributor's fork. 

3. As I use emojis a lot to help me categorize the code review comments (based on [this article](https://github.com/erikthedeveloper/code-review-emoji-guide)), it's slightly annoying that AZDO doesn't auto-complete and understand emojis like `:wrench:`, `:question:`, `:seedling:. To be fair, on Windows, you can use the WIN-dot pop-up as a workaround.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-emojis.png" class="align-center" /> 

3. If you paste a link to another pull request or issue into a pull request, AZDO will just do that: treat it as a link. GitHub is smart enough to realize that it is something native to GitHub and change it into a shortened version of that link that looks like any reference to something within GitHub. In fact, GitHub will also show you a summary of that issue or pull request in a pop-up if you hover your mouse over it.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-hover.png" class="align-center" /> 

## Other features

{:start="14"}
1. As I said earlier, the GitHub C# parser has a deep understanding of the language. And because of that, it can provide a list of symbols within a class on a side panel. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-symbols.png" class="align-center" /> 

    But not only that, when you select a symbol such as the method below, it'll immediatelly show you where that method is defined and where it is used. Not a critical feature, but it has helped me avoid the need to open up an IDE on many occasions. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-references.png" class="align-center" /> 
    
    And if you really need to power of a real IDE in your browser, just press the . (dot) key to open up the repository in [Visual Studio Code for Web](https://code.visualstudio.com/docs/editor/vscode-web). 

1. Although not a big issue, I do prefer the direct connection between a repository and its build pipelines. I do see the advantage of having a pipeline associated with multiple repositories as AZDO has it, but then make it visually clear how to find the pipeline for a repo. I now have to manually add a badge to the read-me.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/azdo-badge.png" class="align-center" /> 

1. One of the biggest maintenance challenges most software projects have these days is to keep up with new versions of the open-source NPM and NuGet packages they depend on. I've seen my fair share of projects that neglected this and ran into breaking changes or hard to solve dependency conflicts at the most inconvenient time. And don't get me started on vulnerabilities introduced by this. In GitHub, has something called [Dependabot](https://github.com/features/security) that will automatically create pull requests that update your Nuget and NPM packages. It's extremely smart, understands semantic versioning and is configurable enough to group updates to related packages. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-dependabot.png" class="align-center" /> 

1. While pushing a new branch from the CLI, GH will give you a link you can click to directly create a pull request on the website. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-cli-pr.png" class="align-center" /> 

1. GitHub introduced a command-line tool called the [GitHub CLI](https://cli.github.com/) that allows you to do practically everything you can through the website, think of things like creating forks, open issues, check out pull requests locally and many others. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-cli.png" class="align-center" /> 

    I use `gh` a lot to review pull requests locally, something which requires quite some git magic on AZDO. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-cli-pr-checkout.png" class="align-center" /> 

1. Another awesome GitHub feature that AZDO doesn't have is the ability to generate release notes from pull requests. In Fluent Assertions, we use that heavily and will result in something like this

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-release-notes.png" class="align-center" /> 

    Notice how it groups the pull requests, adds the links, avatars and names of the contributors and even mentions first-time contributors. And it's all heavily customizable. Just check out [the configuration](https://github.com/fluentassertions/fluentassertions/blob/develop/.github/release.yml) Fluent Assertions uses. 

1. GitHub offers extensive insights in the activity of a repository, something unavailable in AZDO. Especially in larger organization with 100+ repositories, it's a nice way to see which package is still maintained. 

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-insights.png" class="align-center" /> 

1. In GitHub, you can compare any commit, branche or tags. In AZDO, you can only compare tags with tags or branches with branches. This can be quite annoying if you're, for example, trying to figure out what changed between a feature branch and the last released tag. And the URL for comparing is quite human-readable. For example, `https://github.com/fluentassertions/fluentassertions/compare/6.12.0...develop` will give you something like this.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-compare.png" class="align-center" /> 

1. Although AZDO does have some level of syntax highlighting for most file types, GitHub generally supports more file types and has better understanding of C# files. Compare for example the left screenshot from GitHub and a similar one from AZDO. Although the difference isn't that big, you can see that the highlighter in GitHub really understands C#.

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/github-syntax.png" class="align-left" style="max-width: 400px"/> 
    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2024/azdo-syntax.png" class="align-right" style="max-width: 400px"/> 

## Wrap-up
That was a bigger post than I initially thought it would take. Regardless, if you care about clean source control, quick and thorough reviews and intensive collaboration between teams, I urge you to drop Azure DevOps for source control and switch to GitHub. Even more if you care about up-to-date dependencies and reducing the risk of vulnerabilities. IMO, Dependabot is already reason enough to switch. 

GitHub offers such a refined experience, it's just not fair to other source control providers. It uses human-friendly URLs all over the place and its user interface is continuously being improved. In AZDO, it feels like the team is doing the minimum amount of work to support the most requested features and without considering proper interaction design. In GitHub, everything feels so well-thought-out. 

If you're considering to drop AZDO altogether, know that work item tracking is not yet on par with AZDO. Although looking at the pace at which new issue tracking features are added, I'm sure it's getting there pretty fast. And if you can't wait for that and want to completely drop AZDO, switch to JIRA. I have nothing but great experiences with JIRA.

I resent every day I have to work with AZDO. Not because of personal feelings, but because I've seen myself how it is holding back the teams I work with to collaborate more efficiently and commit code that has a high level of traceability. 


## About me
I'm a Microsoft MVP and Principal Consultant at [Aviva Solutions](https://avivasolutions.nl/) with 27 years of experience under my belt. As a coding software architect and/or lead developer, I specialize in building or improving (legacy) full-stack enterprise solutions based on .NET as well as providing coaching on all aspects of designing, building, deploying and maintaining software systems. I'm the author of [Fluent Assertions](https://www.fluentassertions.com), a popular .NET assertion library, [Liquid Projections](https://www.liquidprojections.net), a set of libraries for building Event Sourcing projections and I've been maintaining [coding guidelines for C#](https://www.csharpcodingguidelines.com) since 2001. You can find me on [Twitter](https://twitter.com/ddoomen), [Mastadon](https://mastodon.social/@ddoomen) and [Blue Sky](https://bsky.app/profile/ddoomen.bsky.social).


