---
layout: post
title: Using GitHub Pull Requests as a chat interface to your LLM/Agent
category: llms
---

## Introduction

Lately I have noticed a very alarming trend that affects several open source projects. People use GitHub issues as a chat interface for their agent/llm. 

A person wants to contribute to an open-source project, with the following process:

1. The contributor uses their favorite agent/llm/chat/bot to create the code for the PR
1. They open the PR, and as "description" they just copy-paste the output of the LLM
1. The human maintainer sees the PR and adds comments
1. The contributor just copies-pastes the human comments back to the LLM as a prompt
1. The response and fixes from the LLM are again pasted back to the PR as an comment
1. ...and so on.

What we have here is essentially a standard LLM chat, but in asynchronous mode. The end result is using GitHub issues/PRs/discussion
as a relay service for whatever LLM the contributor is using.

On one hand getting more PRs in your project sounds a like a good problem to have. It means that people are using your project and also that they want to contribute to it. In fact, these people might not be able to contribute at all if they didn't have an LLM at their disposal.

On the other hand, these kinds of PRs (at the time of writing) need a lot of baby sitting and hand-holding. Creating the PR might take less effort for the contributors but increases the effort on behalf of the human maintainers.

I have noticed 3 major problems with llm-generated PRs:

1. PRs that fail review without any hope for salvation
1. PRs that pass review but don't do what they actually say they do
1. PRs that pass review but force the reviewer to act like a robot instead of a human.

## Pull Requests that fail review

This is probably the most straightforward category, but also the one that consumes the most maintainer time. The process goes like this

1. The contributor opens the initial PR
1. The maintainer says that the PR is good but has a single problem i.e. a test does not pass
1. The contributor fixes the test and commits again
1. This time there is linter error
1. The maintainer points out that the test is indeed fixed but the linter error now appeared
1. The contributor commits again. The linter error is fixed
1. The PR checks run again and report that the same (or a different) unit test is broken again 

...and we are back to square one.

Every time I looked at a PR like this something died inside me.

I don't really know what is the problem. Maybe the contributor doesn't know how Git works? Maybe they don't know how to keep the previous context? Maybe they just didn't read my code review comments?

I instantly know that this PR will take too much time for any maintainer to review.

## Pull Requests that pass checks but have architecture issues

The second category of llm generated PRs is also the most dangerous one. A PR that looks
good, passes all checks, everything is green......but does something different.

Common examples are:

1. A PR that implements feature X but has unit tests that verify feature Y 
1. A PR with tests that will always pass regardless of how the feature is implemented
1. A PR that passes everything but also includes changes in **existing** tests

The last case is critical and it is the one that made me write the present article. 

If your PR changes existing tests (i.e. not the ones you added for the new feature) it means that:

1. You changed the previous behavior of the application
1. You just introduced a breaking change for existing users. Oops!

For a human developer, this distinction is pretty obvious. But for an LLM (at least at the time of writing) things are not that simple.

I hear all the time that LLMs will soon (or already do) produce code which will be merged
straight away without any human supervision and I keep asking myself how can this be possible with the current models that we have at our disposal. How many PRs that look good on paper, but are doing something else entirely have already been merged in production code?

## The human factor - 

## Solutions


## Call to action




