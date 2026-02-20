---
layout: post
title: Your "AI" Pull Request was rejected and you don't know why
category: llms
---

## Introduction

If you are reading this, it probably means that an open source maintainer has sent you this blog
and asked you to read it. Maybe it was me after you opened a Pull Request to projects I maintain.

At the time of writing (early 2026) a lot of open source projects are getting bombarded with LLM/agent/AI Pull requests that are either irrelevant, badly designed or even completely wrong.

Several projects [have suffered](https://daniel.haxx.se/blog/2025/07/14/death-by-a-thousand-slops/) from this and open source maintainers are currently trying different approaches such as [rejecting](https://github.com/tldraw/tldraw?tab=contributing-ov-file#making-changes), [restricting](https://matplotlib.org/devdocs/devel/contribute.html#restrictions-on-generative-ai-usage) or even [prohibiting](https://github.com/ghostty-org/ghostty/blob/main/AGENTS.md#issue-and-pr-guidelines) Pull Requests altogether. While [new ideas](https://github.com/mitchellh/vouch) are also on the way, I personally believe that it is better to solve the problem at its root. Instead of combating Pull Requests I would like to educate YOU instead and help you improve yourself.

So if you really want to become a proper Open source contributor read on.

## Part 1 - Creating the Pull Request

Let's focus first on the initial creation of the PR.

Ideally there should already an existing issue for the code that you contribute. **Don't just submit a PR out of the blue**. Issues are great for discussions and you can get early feedback from the maintainers. It will save you (and them) a lot of time (and tokens) if you know before hand that your contribution is relevant to the project.

### Understand the project policy against LLM contributions

If the project you are trying to contribute simply does not accept external contributions, then there is nothing you can do. Move on and try to contribute to another project. The damage is already done (by people like you) and nothing you can say in your Pull Request will convince the maintainers to review/accept it.

### Prime your LLM for best practices 

Several projects already have an `AGENTS` or `CLAUDE` file. For those that don't, try to find the equivalent file that explains how to contribute to the project. This is sometimes called `contributing.md`, other times there is a separate `docs` folder that contains instructions for contributions. It might also be a subset of the `README` file. It is your duty to find this information.

**Make sure that your LLM is aware of all this content**. Verify that at each session this information is always passed to the LLM either as an explicit directive or via the context mechanism your favorite agent is using.

When you think you are finished with your Pull Request again double check that it conforms against contribution guides. Some projects also have a [PR template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository) that you can use to verify if your contribution covers the basics.

### Your PR doesn't actually solve what it is supposed to implement

This might sound basic, but it happens. If you are submitting a PR against an existing issue, make sure that it solves that issue alone. Nothing more and nothing less.

Several times new contributors try to bundle multiple fixes in the same PR. This makes code reviews very difficult on behalf of the maintainers. LLMs can also get carried away and try to sneak into a PR other refactorings or minor features that should be handled in a different PR.

Try to contain your LLM into a single feature at each time. If you want to do major refactorings open a different issue and ideally ask for feedback from the maintainers **BEFORE** you submit a PR.

### Your PR created duplicated code

This is probably the most common problem with your Pull requests. If you don't give enough context to your LLM, it will

 * miss existing methods that can be reused
 * introduce new methods that re-implement existing functionality
 * emit code that doesn't follow the [DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

 You can address this by passing the correct prompts. You should also use the "Plan" mode of you tool where one of the tasks it simply to examine existing code and understand what is there already.

 Example prompt:

 > Before making any changes, examine `common.go` and all files under the `utils` folder to understand what reusable methods already exist in the project and use them accordingly in the fix.

 While several projects have linters in place to detect duplicate code, LLMs can very easily create code snippets that implement existing functionality that doesn't actually look "similar" to existing methods.

### Your PR doesn't have tests

### You implemented the wrong type of tests

### Your PR breaks existing functionality

### Your PR changes existing tests

This 

### You missed corner cases and racing conditions

## Part 2 - Getting feedback after code review

### Don't use PR comments as a relay chat

### Continue working on the PR with the correct context

### Group all feedback in a single session

## Moving forward



