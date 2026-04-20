---
layout: post
title: Your "AI" Pull Request was rejected and you don't know why
category: llms
---

## Introduction

If you are reading this, it probably means that an open source maintainer has sent you this blog post and asked you to read it. Maybe it was me after you opened a Pull Request on projects I maintain.

At the time of writing (early 2026), a lot of open source projects are getting bombarded with LLM/agent/AI Pull Requests that are either irrelevant, badly designed, or even completely wrong.

Several projects [have suffered](https://daniel.haxx.se/blog/2025/07/14/death-by-a-thousand-slops/) from this, and open source maintainers are currently trying different approaches such as [rejecting](https://github.com/tldraw/tldraw?tab=contributing-ov-file#making-changes), [restricting](https://matplotlib.org/devdocs/devel/contribute.html#restrictions-on-generative-ai-usage), or even [prohibiting](https://github.com/ghostty-org/ghostty/blob/main/AGENTS.md#issue-and-pr-guidelines) Pull Requests altogether. While [new ideas](https://github.com/mitchellh/vouch) are also on the way, I personally believe that it is better to solve the problem at its root. Instead of combating Pull Requests, I would like to educate YOU and help you improve.

So if you really want to become a proper open source contributor, read on.

## Part 0 - time is the ultimate constraint

The name of the game is *Time*. Open source maintainers are always short of time. And the more popular an open source project is the more time constrained the maintainers will be.
Maintainers who are faced with several open Pull Requests will work according to the obvious priorities

1. PRs that come from people who have already contributed to the project and are known for creating quality work
1. Simple and minimal PRs that are very easy to understand and evaluate
1. PRs that implement something that has already been described extensively in a GitHub issue and the code part is just the last step

When you submit a brand new PR to an open source project, you automatically fall into the "unknown" bucket. The maintainers do not
really know if you are a coding genius who only uses AI agents for expanding their skills or if you don't know what you are doing and you are wasting their time.

Presenting well written code that aligns with the goals of the project means that you can quickly get from the "unknown" bucket to the "known contributor bucket". 

The worst thing you can do is do the exact opposite of what existing contributors do. If you submit a huge PR that has no corresponding GitHub issue and implements a brand new approach (or even a breaking change), your PR will either be rejected right away, or in the best case scenario
result in a lengthy discussion about the code quality. 


## Part 1 - Creating the Pull Request

Let's focus first on the initial creation of the PR.

Ideally, an issue should already exist for the code you're contributing. **Don't just submit a PR out of the blue.** Issues are great for discussions, and you can get early feedback from the maintainers. It will save you (and them) a lot of time (and tokens) if you know beforehand that your contribution is relevant to the project.

Several times there are multiple open GitHub issues at the same code area. Understand if they are duplicates or different issues. If what you are trying to implement has already been discussed before make sure to pass the discussions/PRs to your AI agent as well.

### Understand the project policy against LLM contributions

If the project you are trying to contribute to simply does not accept external contributions, then there is nothing you can do. Move on and try to contribute to another project. The damage is already done (by people like you), and nothing you can say in your Pull Request will convince maintainers to review or accept it.

### Prime your LLM for best practices

Several projects already have an `AGENTS` or `CLAUDE` file. For those that don't, try to find the equivalent file that explains how to contribute to the project. This is sometimes called `contributing.md`, other times there is a separate `docs` folder that contains instructions for contributions. It might also be a subset of the `README` file. It is your duty to find this information, read it yourself and also pass it to your agent.

**Make sure that your LLM is aware of all this content every time.** Verify that in each session, this information is always passed to the LLM either as an explicit directive or via the context mechanism your favorite agent uses.

Example prompt:

 > Read docs/contributing.md to understand how e2e tests work. Run all e2e tests for feature "x" locally first so that we can see how they behave before making any further changes

When you think you are finished with your Pull Request again, double check that it conforms to contribution guides. Some projects also have a [PR template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository) that you can use to verify if your contribution covers the basics.

### Your PR doesn't actually solve what it is supposed to implement

This might sound basic, but it happens. If you are submitting a PR against an existing issue, make sure that it solves that issue alone. Nothing more and nothing less.

Several times, new contributors try to bundle multiple fixes into the same PR. This makes code reviews very difficult for maintainers. LLMs can also get carried away and try to sneak into a PR other refactorings or minor features that should be handled in a different PR.

Try to contain your LLM to a single feature at a time. If you want to do major refactorings open a different issue and ideally ask for feedback from the maintainers **BEFORE** you submit a PR.

### Your PR created duplicated code

This is probably the most common problem with your Pull Requests. If you don't give enough context to your LLM, it will

 * miss existing methods that can be reused
 * introduce new methods that re-implement existing functionality
 * emit code that doesn't follow the [DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

 You can address this by passing the correct prompts. You should also use the "Plan" mode of your tool where one of the tasks is simply to examine existing code and understand what is there already.

Example prompt:

 > Before making any changes, examine `common.go` and all files under the `utils` folder to understand what reusable methods already exist in the project and use them accordingly in the fix.

While several projects have linters in place to detect duplicate code, LLMs can very easily create code snippets that implement existing functionality that doesn't actually look "similar" to existing methods.

### Your PR doesn't have tests

Most open source projects have a testing policy. They will either reject code that doesn't have tests or reject code that is not tested enough (by looking at code coverage). Make sure that your PR also includes tests.

A classic mistake is creating a test with your AI agent *after* the feature has been implemented. This means that your agent *thinks* this test verifies the new feature. To be certain, you should actually do the opposite:

1. Ask your agent to create the test first
2. Run it and see if it fails
3. Ask the agent to implement the feature
4. Run the test and see if it succeeds

Read more about [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development).

Alternatively, you could also comment out the code yourself that implements the test and see if it fails, if you created the test after the feature was already implemented.

I have personally seen PRs where the tests do not actually verify the feature the associated code is implementing. This is one of the biggest red flags for you when an open source maintainer is going to review your PR. It is one of the fastest ways to lose your credibility with any project.

### You implemented the wrong type of tests

A related problem is when your tests are in the wrong place. Most projects have different types of tests.

1. Unit/fast/method tests
2. Slow/component/e2e/integration tests
3. UI/Functional tests for projects that also have a graphical interface

See the first 3 points on my [testing guide](../testing/software-testing-antipatterns.html) if you want to know about these categories.

Component and end-to-end (e2e) tests are more difficult to set up and run. Your AI agent will probably default to creating the simple unit tests if not guided correctly.

Don't be alarmed if the open source maintainer says that you need an e2e test as well. It probably means that your feature cannot be covered by just unit tests.

Example prompt:

 > Read the e2e tests at folder "x" and locate those that do not test just business logic but check other concerns such as transactions, locking or concurrency issues. Implement a similar test for feature "y" if required.

On the other hand, not every new feature needs e2e tests. Try to understand what your feature needs. Talk with your agent to learn more about the reasoning.

Remember: when submitting a PR you should always be ready to explain *why* something was done this way.


### Your PR breaks existing functionality

Another common issue with AI agents is that they do not understand the impact of their changes. A lot of "guides" explain how you can use an agent in a testing loop where implementing a feature is done when the tests pass correctly. The problem is that making something work by fixing the code is very different than fixing the test. 

It is your responsibility as a human to understand if your PR:

* is only implementing brand new functionality that doesn't break anything for existing users
* it fixes a bug (and test) for something that never worked correctly in the first place
* it updates (and possibly breaks) an existing workflow but this has already been discussed in a GitHub issue and maintainers approve the change.


Example prompt:

 > What happens with existing users of version "x" if feature "y" is implemented? Does it break existing functionality for them?

For most open source projects, backwards compatibility is paramount. This means that even if your PR is perfect on a technical level, but 
breaks current functionality for existing users, maintainers will reject it without some kind of migration plan or documentation warning.


### Your PR changes existing tests

This is a sub case of the previous problem. Unit tests function as runnable specification for a project. Every time your AI agent is updating an existing test, **you are changing expected behavior for current users**. Changing an existing test means changing the specifications.

When you submit a PR you should know the impact of changing existing tests. Don't just blindly let your agent change existing tests just to make a feature "pass". Do some planning to understand *why* the test must change and if it is better to implement something more in the code to keep the existing
test as is.

### You missed corner cases and race conditions

Some open source projects are more complex than others. AI agents are very enthusiastic about implementing the actual business logic of a new 
feature, but they often forget about the "boring" stuff that is required such as error handling, concurrency and backwards compatibility.

For established open source projects, creating a feature that works in 80% of the cases is easy. The problem is getting to 95% or even 100% for all users. If you want to implement a major new feature you need to understand the history of the project, the common guidelines and the code hot-spots that will affect your Pull Request.

Again, planning mode is your friend here. Guide your agent to look not only at the public documentation of the project, but also at existing issues and Pull Requests (even closed ones). The worst thing you can do is create a PR that resurfaces an old issue or misses a very obvious use case (see also the previous section about backwards compatibility).

Example prompt:

 > What happens if two requests that do "x" arrive at the same time? What happens if multiple users do "y" at the same time? Does the code address the scenario when "z" structure/db/array is read and written at the same time?

Be ready to answer questions of concurrency, multi-tenancy, performance and other tricky parts that affect this open source project.
If the maintainer starts asking questions about these topics on the PR, it means that you never addressed them correctly in the initial submission.

## Part 2 - Getting feedback after code review

Okay, so let's say you followed my advice and submitted a clean, minimal, tested, and well-designed PR to your favorite open source project that follows all existing guidelines.

In an ideal world the maintainer would be impressed with your technical skills and instantly merge your PR to the main branch of the project. Congrats! You are now an open source contributor.

If you are reading this guide, though, it means that things are not like this and the maintainer either rejected the PR right away or added additional comments.

### Don't use PR comments as a relay chat

If the maintainer added a comment, do **NOT** just take their comment, put it on your agent and then just paste the result back to the GitHub discussion.

This is actually the main reason that forced me to write this guide. I want to educate you on open 
source contributions. I don't want to use GitHub discussions as another agent chat where I write the "prompt" and you just spend your tokens to give me a "response".

![AI pasted GitHub comments](../../assets/ai-pr-rejected/ai-generated-comments.png)

Believe it or not but several contributors enjoy the social aspect of open source even more than the technical one. They love to spend their time in discussions (with humans) about new ideas, approaches and code architecture. This is how they learn and improve.

If you simply paste your agent output on the GitHub issue

* you dehumanize the discussion as now I know I am speaking to your LLM and not you
* I am not learning anything new myself by hearing a new perspective. I could also paste my own question on my own agent if I really wanted to do that. (hint: most times I might have done this already)
* you show me another red flag that you don't really know what you are doing.

It is okay to ask an agent for extra clarifications and do code research when you don't understand something. But I expect you to understand my comment and formulate an answer that convinces me you know what you are doing.

I cannot speak for all open source developers, but at least for me if you admit that you don't know
how something works and ask for guidance, it actually motivates me to help you.

### Continue working on the PR with the correct context

If your PR gets several comments about different areas of the code, the maintainer expects that anything fixed or addressed will stay fixed or addressed through the end.

Here is one of the worst ways to address maintainer feedback

1. The maintainer posts comments about two different topics/areas
1. You fix topic A
1. There is discussion about how to fix topic B
1. You fix topic B and break A again

Not only is this clear evidence that you are using an AI agent, but it is also a big red flag that you don't understand how context works. At this point, I am seriously thinking about rejecting the PR altogether because it shows to me that you are missing basic LLM foundations before even talking about the business logic of the code itself.

I will not explain here how to manage context for your agent as there are other resources about this and it is best to consult the documentation of your specific tool.

If you are trying to contribute to open source projects, and you don't know how to keep the relevant information in your agent context, you need to stop submitting PRs
and learn your tools first.

### Group all feedback in a single session

Most open source contributors have other things to do apart from reviewing PRs. Either a completely different job or they have their own features to implement. And most times they review PRs in batches or in dedicated triage/review time.

You can help in a number of ways

1. If you are still working on the PR, but haven't addressed all their comments yet, don't ping them until you are actually ready
1. If you have clarification questions, do your research/planning and ask them all at once.
1. If you know how to address their feedback, address all their points as a whole

The worst thing you can do is waste their time by working with many interruptions.

Let's say a maintainer has 3 points of feedback on your PR

1. You only fix the first one and ping them
1. They say that the 2 other points are not addressed yet
1. You add another question for point 2
1. They answer for point 2
1. You implement point 2 and ping them again
1. They add another comment for point 2
1. You leave point 2 and fix point 3 (and ping) them
1. They come to look at the PR and your fix of point 3 also broke something in point 1

**Don't do this**. It wastes time for both parties and makes the whole discussion difficult for anybody to follow.

This is also a red flag because it shows you just enter each individual point into your AI agent instead of presenting a holistic solution (see also the previous point about context).

## Moving forward

I hope this guide has given you some idea of how open source developers think and how you can improve your open source contributions.

At the time of writing, all popular open source projects are suffering from AI Slop Pull Requests that waste their own time (and also tokens). If, by reading this guide, you are no longer part of this problem, I consider that a great win for the whole community.



