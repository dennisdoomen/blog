---

title: How to use GitVersion to get sensible versioning 
excerpt: "How combining the clarity of Semantic Versioning, a release strategy like GitFlow or GithubFlow and GitVersion gives you sensible and automatic artifact numbering"

tags:
- source control
- gitflow
- gitversion
- nuke
- semantic versioning
---

## The challenges of versioning

It's surprising to see that some teams are still struggling with the numbering of their DLLs, NuGet or NPM packages. Some rely on build numbers, some number them by hand by committing a version to source control, and some don't even care about all of this. 

Numbering your artifacts in such a way that they convey a certain level of stability is a matter of maturity that all teams should be adopting. And this goes hand-in-hand with a well-defined branching and merging strategy. In fact, combining the clarity of [Semantic Versioning](https://semver.org/), a release strategy like [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) or [GithubFlow](https://docs.github.com/en/get-started/quickstart/github-flow) and [GitVersion](https://gitversion.net/) gives you all of this practically for free. 

To demonstrate this, let me show you a typical development workflow. So let's assume you have been developing your newest marvel of a software package on a `main` or `master` branch and you've are tagging the first official release as 1.0.0.

    > Git checkout main
    > Git tag 1.0.0

Let's start by installing GitVersion. If you're on Windows and you use @Chocolatey, this is what brings magic to your machine:

    > choco install gitversion.portable --yes

Now run `gitversion` from your console or bash. It'll give you the following output:

    > gitversion

    {
    "Major": 1,
    "Minor": 0,
    "Patch": 0,
    "PreReleaseTag": "",
    "PreReleaseTagWithDash": "",
    "PreReleaseLabel": "",
    "PreReleaseLabelWithDash": "",
    "PreReleaseNumber": null,
    "WeightedPreReleaseNumber": 60000,
    "BuildMetaData": null,
    "BuildMetaDataPadded": "",
    "FullBuildMetaData": "Branch.main.Sha.50a50393d9cd0c9524fd3bb0bea1183c93a0f6c1",
    "MajorMinorPatch": "1.0.0",
    "SemVer": "1.0.0",
    "LegacySemVer": "1.0.0",
    "LegacySemVerPadded": "1.0.0",
    "AssemblySemVer": "1.0.0.0",
    "AssemblySemFileVer": "1.0.0.0",
    "FullSemVer": "1.0.0",
    "InformationalVersion": "1.0.0+Branch.main.Sha.50a50393d9cd0c9524fd3bb0bea1183c93a0f6c1",
    "BranchName": "main",
    "EscapedBranchName": "main",
    "Sha": "50a50393d9cd0c9524fd3bb0bea1183c93a0f6c1",
    "ShortSha": "50a5039",
    "NuGetVersionV2": "1.0.0",
    "NuGetVersion": "1.0.0",
    "NuGetPreReleaseTagV2": "",
    "NuGetPreReleaseTag": "",
    "VersionSourceSha": "50a50393d9cd0c9524fd3bb0bea1183c93a0f6c1",
    "CommitsSinceVersionSource": 0,
    "CommitsSinceVersionSourcePadded": "0000",
    "UncommittedChanges": 0,
    "CommitDate": "2022-01-27"
    }

There's a lot of information in this JSON object, but the one that we will be looking at the most is `FullSemVer`. It shouldn't surprise you that its value is `1.0.0`. 

The rest is meant to be used for versioning .NET assemblies, NuGet packages and NPM packages. I particularly like the `InformationalVersion` which even includes the git hash of the commit from which the number was calculated. A neat trick is that GitVersion understands and detects most build systems and automatically updates the build number reported to TeamCity, Azure Devops, AppVeyor and Github Actions.

## A typical development workflow
Now assuming that your product is relatively new and you haven't reached the confidence level that you can ship every commit to `main` immediately, we will adopt the GitFlow release strategy. This caters for an explicit stabilization phase before you release into production. So let's create a dedicated branch to build the next version on and then re-run GitVersion.

    > git checkout -b develop
    > gitversion /showvariable FullSemVer

    1.0.0

This may surprise you, but the fact is that the `develop` branch is still pointing at the same commit with that `1.0.0` tag. Let's change some files and commit some changes.

    > git add .
    > git commit -m "Some changes on develop"
    > gitversion /showvariable FullSemVer

    1.1.0-alpha.1

GitVersion assumes that new functionality is developed on the `develop` branch, and by default, it assumes it will be backwards compatible. Since the 2nd number in a semantically versioned number represents exactly that, it automatically bumps that version. It also assumes that code on that branch represents the least stable version of the code-base, hence the `alpha` postfix. The `.1` represents the number of commits since the most recent tag on that branch. So adding three more commits will give you:

    > ….commit three more times
    > gitversion /showvariable FullSemVer

    1.1.0-alpha.4

Now imagine you are ready to stabilize your next release for production, but you want to continue developing other features for a future release. This is the point when you start off a release branch. 

    > git checkout -b release/1.1
    > gitversion /showvariable FullSemVer

    1.1.0-beta.1+0

Just like we saw on the `develop` branch, a part of the version increments with each commit on that branch. But since adding `.0` would result in an ambiguous number, `+0` is used. If you read the [relevant section](https://semver.org/#spec-item-10) of the Semantic Versioning spec, you'll find that the numbers after the `+` sign are irrelevant when comparing numbers. Also notice that `release-1.1` and `release/1.1` are equivalent. I prefer using a slash because some Git tools will use that to display the branches in a collapsible folder structure. 

So after having committed some bugfixes to the release branch, you end up with this:

    > gitversion /showvariable FullSemVer

    1.1.0-beta.1+3

The default postfix is `beta`, but I usually change that to `rc` to emphasize the nature of that release. Anyway, let's assume that it's time to release a beta if you will. 

    > git tag 1.1.0-beta.1
    > gitversion /showvariable FullSemVer

    1.1.0-beta.1

No surprise here. You've basically told GitVersion that that last commit represents the first beta now. This is also normally the moment where you up-merge those fixes to `develop`. As soon as you commit a couple more changes to the release branch, this is what will happen.

    > gitversion /showvariable FullSemVer

    1.1.0-beta.2+6

So it understands that you've tagged a commit as beta 1, and assumes everything following that will eventually become beta 2. It does keep counting the commits since the last production release. So your QA team has confirmed the fixes after beta 1 are fine, have approved another beta and agreed to release. 

    > git tag 1.1.0-beta.2
    > gitversion /showvariable FullSemVer

    1.1.0-beta.2

    > git checkout main
    > git merge release/1.1
    > gitversion /showvariable FullSemVer

    1.1.0-beta.2

Because that beta tag is now visible from the `main` branch, that's the version GitVersion will give you now. Normally, this is the point where you tag that same commit as 1.1.0.

    > git tag 1.1.0
    > gitversion /showvariable FullSemVer

    1.1.0

But what does that mean for `develop`? 

    > git checkout develop
    > gitversion /showvariable FullSemVer

    1.2.0-alpha.8

Since that `1.1.0` tag is _not_ visible from the `develop` branch, it might come as a surprise that GitVersion still bumped the minor version. That's the result of GitVersion's `track-merge-target` flag that is enabled in the [default](https://gitversion.net/docs/reference/configuration) configuration. It knew it had to bump the minor part when you upmerged the release branch with the beta tags on it. And even if you merge `main` into `develop` (and you should), nothing will change:

    > gitversion /showconfig

    1.2.0-alpha.8

Now that the release has completed, it is time to delete the release branch.

    > git branch -D release/1.1

## What about hotfixes?
A couple of weeks later, you receive a bug report that affected the 1.1.0 release that is currently living on `main`. Let's create a branch to repair that fix. 

    > git checkout main
    > git branch -D hotfix/1.1
    > gitversion /showvariable FullSemVer

    1.1.0

Again, since nothing was committed yet, the number doesn't change either. Now commit the code fix and try again:

    > git commit -m "some changes"
    > gitversion /showvariable FullSemVer

    1.1.1-beta.1+1

See? GitVersion recognizes the significance of the `hotfix` prefix and automatically bumped the patch number of the version. And just like you saw with a release branch, the `+` symbol is used to track the commits since the last tag. 

After thoroughly testing your fix, you'll release that hotfix by merging it directly into `main` and then tagging the commit. 

    > git checkout main
    > git merge hotfix/1.1
    > git tag 1.1.1
    > gitversion /showvariable FullSemVer

    1.1.1

The cool thing about all of this is that successive hotfixes will automatically bump the version to `1.1.2-beta.x`, even if you keep naming your hotfix branch `hotfix/1.1`. Oh, and don't forget to up-merge all your fixes from `main` to `develop` as well. You don't want to run the risk of fixing something on `main` that regresses after the next major and minor release.

## Time to prepare for that next major release
Now say that the work that continued on the `develop` branch involves major functionality that requires breaking changes. How would you tell GitVersion that it should apply a major bump instead of the usual minor bump? Let's look at the default configuration that GitVersion uses.

    > gitversion /showconfig

    assembly-versioning-scheme: MajorMinorPatch
    assembly-file-versioning-scheme: MajorMinorPatch
    mode: ContinuousDelivery
    tag-prefix: '[vV]'
    continuous-delivery-fallback-tag: ci
    major-version-bump-message: '\+semver:\s?(breaking|major)'
    minor-version-bump-message: '\+semver:\s?(feature|minor)'
    patch-version-bump-message: '\+semver:\s?(fix|patch)'
    no-bump-message: '\+semver:\s?(none|skip)'
    legacy-semver-padding: 4
    build-metadata-padding: 4
    commits-since-version-source-padding: 4
    tag-pre-release-weight: 60000
    commit-message-incrementing: Enabled
    branches:
    develop:
        mode: ContinuousDeployment
        tag: alpha
        increment: Minor
        prevent-increment-of-merged-branch-version: false
        track-merge-target: true
        regex: ^dev(elop)?(ment)?$
        source-branches: []
        tracks-release-branches: true
        is-release-branch: false
        is-mainline: false
        pre-release-weight: 0
    main:
        mode: ContinuousDelivery
        tag: ''
        increment: Patch
        prevent-increment-of-merged-branch-version: true
        track-merge-target: false
        regex: ^master$|^main$
        source-branches:
        - develop
        - release
        tracks-release-branches: false
        is-release-branch: false
        is-mainline: true
        pre-release-weight: 55000
    release:
        mode: ContinuousDelivery
        tag: beta
        increment: None
        prevent-increment-of-merged-branch-version: true
        track-merge-target: false
        regex: ^releases?[/-]
        source-branches:
        - develop
        - main
        - support
        - release
        tracks-release-branches: false
        is-release-branch: true
        is-mainline: false
        pre-release-weight: 30000
    feature:
        mode: ContinuousDelivery
        tag: useBranchName
        increment: Inherit
        prevent-increment-of-merged-branch-version: false
        track-merge-target: false
        regex: ^features?[/-]
        source-branches:
        - develop
        - main
        - release
        - feature
        - support
        - hotfix
        tracks-release-branches: false
        is-release-branch: false
        is-mainline: false
        pre-release-weight: 30000
    pull-request:
        mode: ContinuousDelivery
        tag: PullRequest
        increment: Inherit
        prevent-increment-of-merged-branch-version: false
        tag-number-pattern: '[/-](?<number>\d+)'
        track-merge-target: false
        regex: ^(pull|pull\-requests|pr)[/-]
        source-branches:
        - develop
        - main
        - release
        - feature
        - support
        - hotfix
        tracks-release-branches: false
        is-release-branch: false
        is-mainline: false
        pre-release-weight: 30000
    hotfix:
        mode: ContinuousDelivery
        tag: beta
        increment: Patch
        prevent-increment-of-merged-branch-version: false
        track-merge-target: false
        regex: ^hotfix(es)?[/-]
        source-branches:
        - develop
        - main
        - support
        tracks-release-branches: false
        is-release-branch: false
        is-mainline: false
        pre-release-weight: 30000
    support:
        mode: ContinuousDelivery
        tag: ''
        increment: Patch
        prevent-increment-of-merged-branch-version: true
        track-merge-target: false
        regex: ^support[/-]
        source-branches:
        - main
        tracks-release-branches: false
        is-release-branch: false
        is-mainline: true
        pre-release-weight: 55000
    ignore:
    sha: []
    commit-date-format: yyyy-MM-dd
    merge-message-formats: {}
    update-build-number: true

Looking at the configuration, you should be able to deduce that you can add a `+semver: breaking` or `+semver: major` to the commit message to bump the version. But there are other ways as well such as adding a `next-version` field to the `gitversion.yaml` file in the root of your repo, or just using a tag to force a number. Alternatively, you could decide to postpone bumping the version until you create a release branch with the appropriate number:

    > git checkout develop
    > gitversion /showvariable FullSemVer

    1.2.0-alpha.8

    > git checkout -b release/2.0
    > gitversion /showvariable FullSemVer

    2.0.0-beta.1+0

So even though GitVersion may number your `develop` branch as `1.2.x`, as soon as you create a release branch with a different number, it'll use that one. 

Let's finish the major release.

    > git checkout main
    > git merge release/2.0
    > git tag 2.0.0
    > gitversion /showvariable FullSemVer

    2.0.0

## What about supporting the previous major version
As we like to follow semantic versioning, the implied consequence of bumping the major part of the version to 2 means that existing users of the package cannot update to that version without some kind of rework, significantly changed behavior, or new technical prerequisites. To keep supporting those users, you'll need another branch:

    > git checkout 1.1.1
    > git checkout -b support/1.x
    > gitversion /showvariable FullSemVer

    1.1.1

No surprise there. As long as there are no fixes needed on a `support` branch, that branch points to the last 1.x.x tag. But as soon as you start pushing commits to that branch, either directly or through a hotfix branch, the number will increase.

    > git commit 
    > gitversion /showvariable FullSemVer

    1.1.2+1

The rest works exactly as on `main`. So in essence, a support branch is nothing like an alternative reality of `main` that starts off when your `main` receives breaking changes. But don't forget to upmerge those fixes back to `main`, any release branches and `develop`. And if you no longer have to maintain version 1, delete the support branch. 

## How do you use GitVersion in your build script
As you saw in the beginning of this article, GitVersion is available through Chocolatey, but also as a `dotnet` global tool, Homebrew and through [various other channels](https://gitversion.net/docs/usage/cli/installation). The way I prefer to use it is as part of a [Nuke build script](https://nuke.build/). It comes out of the box with the default template and adds a field that gets initialized automatically:

```csharp
[GitVersion(UpdateBuildNumber = true, Framework = "net6.0")]
readonly GitVersion GitVersion;
```

Nuke relies on the `GitVersion.Tool` being referenced as a `<PackageDownload>` element in the C# project file. With that, you can use the `GitVersion` field like this:

```csharp
TargetCompile => _ => _
    .Executes(() =>
    {
        DotNetBuild(s => s
            .SetProjectFile(Solution)
            .SetAssemblyVersion(GitVersion.AssemblySemVer)
            .SetFileVersion(GitVersion.AssemblySemFileVer)
            .SetInformationalVersion(GitVersion.InformationalVersion));
    });
```

GitVersion tries to fetch information from other branches as well, so you need to make sure that your build server ensures a [non-shallow clone](https://gitversion.net/docs/reference/requirements) of your git repository. You can find specific instructions for specific build servers [here](https://gitversion.net/docs/reference/build-servers/). 

## What if I force-push changes to my pull request?
This is an interesting case that I've encountered as well. If you follow my [guidance]({% post_url 2020/2020-03-18-keep-source-control-history-clean %}) on delivering a clean pull request for review, you'll be doing lots of interactive rebases and force pushes. In fact, force pushing to a personal feature branch is the _only_ place where you should use a force push. But that means that the number of commits in that PR won't change even if the contents did. If you build NuGet or NPM packages or deploy environments from that PR, it might be desirable to still get a unique number per build. 

In Fluent Assertions, which is build by Github Actions, I've [solved this](https://github.com/fluentassertions/fluentassertions/blob/develop/Build/Build.cs#L59) by adding the following target:

```csharp
[Parameter("A branch specification such as develop or refs/pull/1775/merge")]
readonly string BranchSpec;

[Parameter("An incrementing build number as provided by the build engine")]
readonly string BuildNumber;

string SemVer;

Target CalculateNugetVersion => _ => _
    .Executes(() =>
    {
        SemVer = GitVersion.SemVer;
        if (IsPullRequest)
        {
            Serilog.Log.Information(
                "Branch spec {branchspec} is a pull request. Adding build number {buildnumber}",
                BranchSpec, BuildNumber);

            SemVer = string.Join('.', GitVersion.SemVer.Split('.').Take(3).Union(new [] { BuildNumber }));
        }

        Serilog.Log.Information("SemVer = {semver}", SemVer);
    });

bool IsPullRequest => BranchSpec != null && BranchSpec.Contains("pull", StringComparison.InvariantCultureIgnoreCase);
```

The `BranchSpec` and `BuildNumber` configuration settings are passed by GitHub through the following YAML definition:

```yaml
- name: Run NUKE
    run: ./build.ps1
    env:
    BranchSpec: ${{ github.ref }}
    BuildNumber: ${{ github.run_number}}
```

Then, in my `Pack` target I use the `SemVer` field instead of the one provided by the `GitVersion.SemVer` field.

```csharp
Target Pack => _ => _
    .DependsOn(ApiChecks)
    .DependsOn(TestFrameworks)
    .DependsOn(UnitTests)
    .DependsOn(CalculateNugetVersion)
    .Executes(() =>
    {
        DotNetPack(s => s
            .SetProject(Solution.Core.FluentAssertions)
            .SetOutputDirectory(ArtifactsDirectory)
            .SetConfiguration("Release")
            .SetVersion(SemVer));
    });
```

Given a [customized](https://github.com/fluentassertions/fluentassertions/blob/develop/GitVersion.yml) `GitVersion.yaml` file in that repo, this gives me numbers like `6.0-pr1807.282` where `1807` is the PR number and `282` the build number. 

## Wrap-up
So what do you think? Did I convince you to mature your build process by introducing GitVersion? If not, what then are you planning to use? Let me know by commenting below. Oh, and follow me at [@ddoomen](https://twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions


