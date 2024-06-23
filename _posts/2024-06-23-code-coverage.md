---
layout: post
title: Getting 100% code coverage doesn't eliminate bugs
category: testing
---

TL;DR - Getting 100% coverage on a project doesn't mean you have zero bugs. Here is an extreme example to prove it.

## Introduction

I am still finding a lot of people who want to increase code coverage to unrealistic numbers or who think that getting 100% code coverage is the holy grail that will eliminate all bugs and produce the perfect software.

There are many articles already on the net explaining why this is a fallacy, but I recentrly discovered that sharing an actual code example goes a long way towards proving why 100% code coverage doesn't mean zero bugs. These people have their "aha" moment when they look at real code, instead of recycling theoretical arguments over and over.

So instead of explaining the same example again and again, I am sharing it in this post.

## An "application" with just 1 line of code

I am writing a new application that has exactly [one line of code](https://github.com/kkapelon/code-coverage-is-overrated/blob/main/velocity.go).

```go
func calculateVelocity(angle int, direction int) int {
	return ((3 * (4*angle - direction)) * 3) / (7 * (direction - (2 * angle))) * -1
}
```

This is a super important method that is going to be used for software that launches rockets into space. It needs to be rock solid.

Since I am a perfectionist, I am writing not one, not two but [6 unit tests](https://github.com/kkapelon/code-coverage-is-overrated/blob/main/velocity_test.go) for this function. 
And after running the tests I see that I have achieved the holy grail - 100% code coverage!


![Achieving 100% code coverage](../../assets/code-coverage/coverage.png)

I go to bed and then next morning I see the news about the rocket explosion. Apparently this 
function has a bug and if the direction is double the angle, the functions throws a divide-by-zero error.

## FAQ

Q. This article is useless. Everybody knows that 100% code coverage doesn't eliminate bugs.  
A. Junior developers [certainly don't know this](https://xkcd.com/1053/). Misguided team managers don't know it as well. I actually wrote this article, because now I can give everybody the link instead of showing the same example again and again.

Q. Where did you got the numbers from your unit tests?  
A. From my business analyst, or my QA engineer or my buddy Fred. Does it really matter? All it matters
is that I had 100% code coverage with them.

Q. Of course it matters. As soon as I saw your method I thought about the error and I would write a unit test for that. Your QA engineer missed that case.  
A. Sure you can do that for 1 line of code. But you can do it for 5? for 500? for 50,000? If yes, then you have a superhuman ability and a) I want to hire you b) no need to write unit tests any more. Just fix the code right away if you can spot all bugs or a system.

Q. So you are saying that unit tests are useless?  
A. No, of course not. Tests are great for catching regressions or functioning as written specifications. All I am saying is that trying to increase code coverage in order to have less bugs is a losing battle.

Q. So if code coverage is not a good metric for the quality of the project, what metrics should I look instead ?  
A. I have suggested the useful but largely unknown metrics `PDWT`,`PBCNT`,`PTVB`,`PTD` in [my testing guide]({{site.baseurl }}/testing/software-testing-antipatterns.html#anti-pattern-6---paying-excessive-attention-to-test-coverage).


Q. So what should I do if I want less bugs on my mission critical software?  
A. You should look at other approaches such as [Formal verification](https://en.wikipedia.org/wiki/Formal_verification) and [proof checkers](https://en.wikipedia.org/wiki/Proof_assistant).

Q. Look this is great, but I still need to find the perfect code coverage for my new project. If it is not 100%, what should I aim for?  
A. Without more context on your project all I can say is that the minimum code coverage is 20%. This number is based on the [Pareto Principle](https://en.wikipedia.org/wiki/Pareto_principle) and assumes that 20% of your code is responsible for 80% of your bugs. See also my point about "critical code" in [my testing guide]({{site.baseurl }}/testing/software-testing-antipatterns.html#anti-pattern-4---testing-the-wrong-functionality).

Q. My organization/manager/customer insists on achieving 70%/80%/100% code coverage on all projects. What should I do?   
A. Send them this article. If you cannot convince them, maybe try to find a different organization/company/customer to work for?