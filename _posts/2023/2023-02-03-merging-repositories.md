---

title: "Merging Git repositories without losing history"
excerpt: "With a bit of Git magic, it's quite easy to merge the commits of one repository into a specific folder of another without losing the Git history"

tags:
- git
- 
- building
---

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/2023/merging.jpg" class="align-center"/> 

I'm the first to admit that merging two repositories isn't something that you would do often. On the other hand, I've seen my fair share of situations where teams were keeping separate repositories for their front-end SPA application and the accompanying back-end APIs and running into issues. Even though keeping them separate may give a sense of isolation, in reality you end up coordinating teams across those repos whenever a big feature needs to introduced. And don't get me started on the challenges getting those HTTP APIs aligned. So moving to a mono-repo isn't that uncommon after all. 

With a bit of Git magic, it's quite easy to merge the commits of one repository into a specific folder of another without losing the Git history. Here's a high-level overview. 

1. Install `git-filter-repo` using the instructions Elijah Newren prepared on [his repo](https://github.com/newren/git-filter-repo#how-do-i-install-it). 
1. As the `git filter-repo` command is a destructive operation, clone the source repo to a safe location
1. Determine the directory at which the code from the source repo should end up
1. Determine the prefix you want to use to add to all tags from the source repo
1. Run `git filter-repo`
1. Add the modified working directory as a git remote to the target repository
1. Run a merge that allows unrelated histories

So let's assume you have a repo named `Source` that you want to merge in the `Target` repo. Also suppose you want the files from `Source` to end up in `Target` under the directory `Sources/Foo`. Let's start by creating a fresh clone of `Source`. 

    cd workspaces
    git clone git@github.com:dennisdoomen/Source.git Source
    cd source

The first step is to rewrite the history of that repo so that all files appear to be in the `Sources/Foo` directory  and prefix all existing tags with `foo-`. Assuming our code is on the `master` branch, we can do that using:

    git checkout master
    git filter-repo --to-subdirectory-filter Sources/Foo --tag-rename :foo-

If everything worked as expected, you should now see that all files were moved to a Source/Foo directory.

Now we can move to the `Target` repo and add `Source`'s working directory as a new remote. So assuming that the Target repo is in the same `workspaces`:

    cd ..\target
    git remote add -f local ..\source
	
The final step is to import the commits from that new remote into the current one, ignoring the fact that they have no common history. And remember, we're importing the `master` branch here.

	git merge --allow-unrelated-histories local/master

Since the `git filter-repo` command rewrote the Source repo, we should delete that clone. 

That's it. That's all it takes to merge repositories. And this works equally well when merging repositories that were integrated using a submodule. In fact, that's exactly why we needed to figure this out in the first place. 

So what do you think? Did you ever have to do this and found an alternative. Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
