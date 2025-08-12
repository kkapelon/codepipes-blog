---
layout: post
title: Why Your Next App Will Probably Suck Without…Unit Testing
category: testing
---

_This post was originally published at the [JRebel/Zeroturnaround blog](https://www.jrebel.com/). Reposting here since it is not available there any more._

## Introduction

Ah, unit testing.

Believe it or not, this is actually a controversial topic among Java developers. Some think that writing unit tests is a waste of time, while others consider them an essential part of the software development process, and the way to ensure that your new app doesn’t, well, suck because it wasn’t checked properly. And yet some developers are bored by the topic of unit tests.

“Why are you even writing about that?” is something I heard recently from a developer friend, who had been using unit tests for so long that he considered them part of the “room” already for Java development.

Put simply, developers use unit tests as an internal control on the functionality and compatibility of their applications when changes to features, code or the environment happen.

Unit tests themselves are composed of programming code (just like the rest of the application), but they are not shipped along with the product. Instead, we use unit tests during development.

In this post, we’ll introduce unit tests to less experienced developers and remind the more senior Java masters out there that it’s still important to review the basics from time to time.

Our goal is to help you learn why unit tests are important to the software development process, plus how to set up and run your own unit tests. But before we do any actual unit testing on your code, let’s review why unit tests are needed.

## Solving problems, saving time and eliminating waste

Use your imagination for a minute to consider this common scenario for companies that haven’t yet integrated unit testing methodologies into their software development process.

* Stage 1: The Operations team is currently running your coolest application, SuperBlagTM v5.3, in production while the development team is busy preparing v5.4.
* Stage 2: After two frantic months of development, the QA team (if it exists in the organization) gets the new version for testing.
* Stage 3: The QA team, naturally, checks the new features of 5.4 (assuming that everything else works correctly).
* Stage 4: After approval by QA, the new version 5.4 is deployed to production.

Sounds pretty reasonable, right? But how familiar does this next part sound?

A few days after releasing v5.4, users are surprised to find that what they want to do (and/or what they USED to do) completely fails. They are furious because this feature worked in version 5.3. The overworked QA team admits that they tested only new functionality for 5.4, and now the dev team is scrambling to discover the bug.

After some investigation, the development team realizes that some changes for the new version have affected more core parts of the application.

They fix the bug and inform QA that, actually, they really need to test EVERYTHING from the very beginning.

The result: users are unhappy and not trusting your app, QA is unhappy with the extra effort and development team is afraid of refactoring.

If we are lucky, the fix is made and a bug-free v5.3.1 reaches production. However there is the possibility that the fix breaks something else in another feature of SuperBlagTM that is found again by your unhappy users after a few more days. Thus we see this endless circle that can become a reality in really big enterprise applications.

This is what unit tests are attempting to solve — if you can solve this problem by catching regressions in application code, then all the things that worked in the previous version should also work in the next.

## Get your hands dirty: “Hello World” in Unit Testing

In Java there is much love for a highly popular unit testing framework called [JUnit](https://junit.org/). Most Java IDEs (i.e. Eclipse, IntelliJ IDEA, NetBeans) have explicit support for JUnit, making it ideal for those of you who are interested in getting started with unit testing.

For the sake of this example, let’s say that the “Enterprise Product” is an e-shop and we want to test the module that calculates the total weight of an order. [Here is the sample code](https://github.com/kkapelon/java-unit-testing-intro/blob/main/src/main/java/com/codepipes/BasketWeightCalculator.java):

```java
public class BasketWeightCalculator {

    private int totalWeight = 0;

    public void addItem(int itemWeight) // Assume weight is always an integer number
    {
        totalWeight = totalWeight + itemWeight;
    }

    public int getTotalWeight() {
        return totalWeight;
    }
}
```

The code just adds the weight of any new item to the existing weight of the basket. Now it’s time to write the unit test.

The suggested way of writing unit tests is to do it gradually. You start with very basic assumptions and continue to more advanced ones. The first thing to test, of course, is a simple case of using just one or two items.

We expect the weight of the basket to be just the sum. But let’s think of a more advanced (yet still simple) test – why don’t we make sure that the order of the items does not matter:

```java
public class BasketWeightTest {

    @Test
    public void oneItem() {
        BasketWeightCalculator weightCalculator = new BasketWeightCalculator();
        weightCalculator.addItem(5);

        assertEquals("Expected 5", 5, weightCalculator.getTotalWeight());
    }

    @Test
    public void twoItems() {
        BasketWeightCalculator weightCalculator = new BasketWeightCalculator();
        weightCalculator.addItem(5);
        weightCalculator.addItem(13);

        assertEquals("Expected 18", 18, weightCalculator.getTotalWeight());
    }

    @Test
    public void orderDoesNotMatter() {
        BasketWeightCalculator weightCalculator1 = new BasketWeightCalculator();
        weightCalculator1.addItem(5);
        weightCalculator1.addItem(13);

        BasketWeightCalculator weightCalculator2 = new BasketWeightCalculator();
        weightCalculator2.addItem(13);
        weightCalculator2.addItem(5);

        assertEquals("Expected 18", 18, weightCalculator1.getTotalWeight());
        assertEquals("Expected 18", 18, weightCalculator2.getTotalWeight());
    }
}
```

This unit test is normal Java code, but there are three points here to consider:

1. The name of the class is “name of the class being tested” + “Test” This is not a strict requirement but it is a good convention followed by several tools.
1. The test methods are annotated with the `@Test` annotation, but the test methods themselves can have any name.
1. The most crucial step is the `assert` Method, found at the end of each test method. This indicates whether the test fails or not.

There are several assert methods for each basic Java type but their syntax is similar. The second argument is the expected result, while the final argument is the actual result. If those two match, then the test is a success. If not the test fails.

Now you can save this file in your source folder and since we are using Eclipse for this example, you can right-click on it and select “run as unit test”. If everything goes ok you will see the infamous green bars!

![Unit test success](../../assets/java-unit-test-intro/first-success.png)

Congratulations! Everything seems to be in order, and your code is matching expectations.
