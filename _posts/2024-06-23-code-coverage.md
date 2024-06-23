---
layout: post
title: Getting 100% code coverage 
category: testing
---

TL;DR - Getting 100% coverage on a project doesn't mean you have zero bugs. Here is an extreme example to prove it.

## Introduction

## FAQ

Q. Where did you got the numbers from your unit tests?  
A. From my business analyst, or my qa engineer or my buddy Fred. Does it really matter?

Q. Of course it matters. As soon as I saw your method I thought about the error and I would write a unit test for that. Your QA engineer missed that case.  
A. Sure you can do that for 1 line of code. But you can do it for 5? for 500? for 50,000? If yes, then you have a superhuman ability and a) I want to hire you b) no need to write unit tests any more. Just fix the code right away

Q. So if code coverage is not a good metric for the quality of the project, what metrics should I look instead ?  
A. I have suggested the useful but largely unknown metrics `PDWT`, `PBCNT`, `PTVB`, `PTD` in [my testing guide]({{site.baseurl }}/testing/software-testing-antipatterns.html#anti-pattern-6---paying-excessive-attention-to-test-coverage).

Q. So you are saying that unit tests are useless?  
A. No, of course not. Tests are great for catching regressions or functioning as written specifications. All I am saying is that trying to increase code coverage in order to have less bugs is a losing battle.

Q. Look this is great, but I still need to find the perfect code coverage for my new project. If it is not 100%, what should I aim for?  
A. Without more context on your project all I can say is that the minimum code coverage is 20%. This number is based on the [Pareto Principle](https://en.wikipedia.org/wiki/Pareto_principle) and assumes that 20% of your code is responsible for 80% of your bugs. See also my point about "critical code" in [my testing guide]({{site.baseurl }}/testing/software-testing-antipatterns.html#anti-pattern-4---testing-the-wrong-functionality).

Q. My organization/manager/customer insists on achieving 70%/80%/100% code coverage on all projects. What should I do?   
A. Send them this article. If you cannot convince them, maybe try to find a different organization/company/customer to work for?