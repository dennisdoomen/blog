---

title: "Why I gave the C# Coding Guidelines a major refresh"
excerpt: "A substantial update to the C# Coding Guidelines adds stronger foundations, a dedicated testability section, better coverage of modern C#, and a healthier amount of pruning."
tags:
---

I've been maintaining the C# Coding Guidelines since 2001. Over the years, I've added, rewritten and removed rules whenever the language, tooling and the way we build software changed enough to justify it.

This is one of the bigger refreshes in a while. Not because I wanted to make the guidelines bigger, but because I wanted them to become more useful again. A guideline collection like this should help teams make better decisions. If it turns into a dusty catalog of old opinions, it loses that value quickly.

So what changed?

## The guidelines now start with stronger foundations

One of the most important additions is a brand-new [General Guidelines](https://csharpcodingguidelines.com/general-guidelines/) section. For a long time, the document jumped rather quickly into class design, member design and low-level coding decisions. Useful, yes, but it missed some of the principles that explain *why* many of those rules exist in the first place.

That gap is now covered by rules around boundaries, design patterns, composition, the Principle of Least Surprise, YAGNI, DRY within boundaries, the three pillars of object-oriented programming, and a new reminder that AI-generated code is still your responsibility.

I particularly like the emphasis on boundaries. Too many teams apply principles such as DRY without asking whether the duplication crosses a meaningful boundary. A tiny bit of duplication inside a bounded context is often less harmful than introducing coupling between modules, services or libraries that should evolve independently.

And yes, I deliberately included guidance on AI. Teams are embracing tools such as GitHub Copilot and other coding agents at a rapid pace, so pretending that guideline collections can ignore that reality would be silly. But the rule is intentionally simple: use AI if it helps, but review, understand and own every line that ends up in your codebase. "The AI wrote it" is not a valid defense in a code review.

In other words, this refresh is not just about adding more rules. It is also about making the document more opinionated where that helps teams reason about architecture and maintainability.

## Testability is now treated as a first-class concern

Another major change is the addition of a dedicated [Testability Guidelines](https://csharpcodingguidelines.com/testability-guidelines/) section. That was overdue.

If you've followed my writing for a while, you already know I don't see testability as some secondary quality attribute. It is one of the strongest indicators that code is readable, well-structured and safe to change. So it never felt right that the guidelines only touched on testing indirectly.

The new testability section fixes that with rules such as:

- use short, functional test names;
- prefer `Specs` over `Tests` for test classes;
- test observable behavior rather than private implementation details;
- prefer inline literals over constants in tests when that improves readability;
- test reusable components separately from the consumers that happen to use them.

That last one matters a lot. Reusable validators, serializers, domain services and extension methods deserve their own focused test suites. If they are only covered indirectly through a higher-level component, failures become harder to understand and refactoring becomes riskier than necessary.

## The document now does a better job covering modern C#

C# has evolved a lot in recent years, and the guidelines needed to catch up. This refresh adds several rules that reflect the language most teams are actually using today.

There is now guidance on when to use a `record` versus a `class`, when a delegate is a better fit than an interface with a single member, and when primary constructors improve readability instead of just saving a few lines. I also added guidance on extension members, required properties, deconstruction, and the new C# 14 `field` keyword for auto-properties with logic.

Those are not cosmetic additions. Language features influence design, readability and maintainability. If a guideline collection ignores that, it quietly encourages outdated habits.

At the same time, I also updated existing rules. For example, some rules now explain concepts more clearly, some examples were modernized, and some topics got extra nuance around abstraction levels, async naming, polymorphism and documentation intent. That kind of maintenance is less visible than adding a shiny new rule, but it is just as important.

## Pruning old rules is just as important as adding new ones

A good set of guidelines should be curated, not hoarded.

That is why this refresh does not only add content. It also removes a fair number of rules that no longer pull their weight. Several older rules disappeared entirely, including some around nested loops, documentation details, framework usage and other topics that were either too narrow, superseded by better guidance elsewhere, or simply no longer important enough to deserve a dedicated rule.

I feel strongly about that. If you never remove anything, readers stop trusting the document's editorial judgment. Every rule should justify its existence. If it cannot, it should be merged, rewritten or deleted.

So yes, the document got bigger, but it also got sharper.

## The site structure finally reflects the content better

Content changes like these need some structural support. So next to the rules themselves, I also updated the introduction, navigation and a few supporting pages.

The introduction now does a better job explaining what the document is, why teams would use it, how to get started, and how it relates to things such as analyzers, agent skills and modern tooling. Navigation was updated to expose the new categories properly, and the resources page and a few styling-related pieces were refreshed along the way.

That last part matters more than it may seem. The repository now points readers to install the accompanying Skills so their preferred AI agent can use the guidelines during code reviews. I like that a lot because it turns the guidelines into something more operational. Instead of being a PDF people vaguely agree with, they can become part of the actual review workflow.

These may look like minor site tweaks, but they matter. A guideline library only helps people if they can find their way around it and understand how to adopt it in practice.

## Why this matters

I don't maintain these guidelines to create a giant list of opinions about semicolons and braces. I maintain them because teams need practical help making trade-offs around readability, coupling, testability, maintainability and modern C# usage.

This refresh brings the guidelines closer to how I currently think about software quality:

- start with principles, not isolated rules;
- treat testability as a design concern;
- embrace modern language features, but only when they improve clarity;
- keep pruning anything that no longer adds enough value.

If you're already using the guidelines, I would recommend starting with the new General and Testability sections before diving into the individual rule categories. And if you've never used them before, this is probably the best time to take another look at [csharpcodingguidelines.com](https://csharpcodingguidelines.com/).

As always, feedback is welcome. A guideline collection like this only stays relevant if it keeps evolving with the language, the tooling and the way we actually build software.
