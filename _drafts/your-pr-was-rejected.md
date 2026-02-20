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

## Understand the project policy against LLM contributions

If the project you are trying to contribute simply does not accept external contributions, then there is nothing you can do. Move on and try to contribute to another project. The damage is already (by people like you) and nothing you can say in your Pull Request will convince the maintainers to review/accept it

## Your PR doesn't actually solve what it is supposed to implement

This might sound basic, but it happens. If you are submitting a PR against an existing issue, make sure that it solves that issue alone. Nothing more and nothing less.

Several times new contributors try to bundle multiple fixes in the same PR. LLMs can also get carried away and try to sneak into a PR other refactorings or minor features that should be handled in a different PR.

Try to contain your LLM into a single feature at each time. If you want to do major refactorings open a different issue and ideally ask for feedback from the maintainers **BEFORE** you submit a PR.

## Your PR created duplicated code

## Your PR doesn't have tests

## You implemented the wrong type of tests

## Your PR breaks existing functionality

## Your PR changes existing tests

## You missed corner cases and racing conditions

## 



