---
layout: post
title: Don’t Test Blindly - The Right Methods for Unit Testing Your Java Apps
category: testing
---

_This post was originally published at the [JRebel/Zeroturnaround blog](https://www.jrebel.com/). Reposting here since it is not available there any more._

## Introduction

In my previous blog post [Why Your Next App Will Probably Suck Without…Unit Testing](https://blog.codepipes.com/testing/java-unit-test-intro.html), we gave an overview promoting the benefits of unit tests, and how these small, simple actions can make a big difference in getting bug-free, solidly-built applications out the door.

After reading this, I can assume that you have jumped out of your chair and started frantically writing them for your big enterprise project, right? Well, stop now!

Before writing a single unit test, it’s necessary to determine exactly what to test. A big enterprise application can hold a billion lines of code, so do you think it’s realistic to write tests for everything? Not exactly. So, it’s important to make sure your tests are focused on what actually matters to you, your app and your users.

## What you should NOT test

You can spend your entire life testing away if you like, but it’s probably best to to make some assumptions in the beginning to save yourself some time. After all, if you end on testing your own database all the time, then you’ve got bigger problems than error-free app deployment. So before explaining where you should focus your unit testing efforts, let’s see what you should NOT test and get it out of the way first.

I’ll ask you to trust my 8 years in software development when I say you should NOT write units tests for:

1. Other framework libraries (you should assume they work correctly)
1. The database (you should assume it works correctly when it is available)
1. Other external resources (again you assume they work correctly when available)
1. Really trivial code (like getters and setters for example)
1. Code that has non deterministic results (Think Thread order or random numbers)
1. Code that deals only with UI (e.g. Swing toolkit, Wicket)

Funny story: When I was just starting to get into code and learned about unit tests, the first thing I did was write a test that saved an Entity on the database using Hibernate, read it back and then verified that it is the same. Turns out that writing unit tests against your own database or Hibernate is not very productive. Awesome :-/

## What you SHOULD test

One of the golden rules of unit testing is that your tests should cover code with “business logic”. Here is the typical flow in a back-end processing system. Your web application could look like this:

![Business logic](../../assets/test-coverage/business-logic.png)

In this case, the highlighted part in gold is where you should focus your testing efforts. This is the part of the code where usually most bugs manifest. It is also the part that changes a lot as user requirements change since it is specific to your application.

So what happens if you get across a legacy application with no unit tests? What if the “business logic” part ends up being thousands of lines of code? Where do you start?

In this case you should prioritize things a bit and just write tests for the following:

1. Core code that is accessed by a lot of other modules
1. Code that seems to gather a lot of bugs
1. Code that changes by multiple different developers (often to accommodate new requirements)

How much of the code in these areas should we test, you might ask. Well, now that we know which areas to focus on, we can now start to analyze just how much testing we need to feel confident about our code.

## Code coverage: How much is enough/too much?

A useful metric that shows what part of your code is “touched” by unit tests is Code Coverage. Code coverage is the percentage that shows to what depth your internal checks affect your Java classes. To obtain this metric, you can run one of multiple tools available for this purpose.

For you Eclipse users out there, an easy way to find code coverage is to install the [Eclemma plugin](https://www.eclemma.org/) from the Eclipse Marketplace. In [our previous post on unit tests](https://blog.codepipes.com/testing/java-unit-test-intro.html), we ran tests on a simple calculation for determining the weight of basket or shopping cart for an e-shop, so we will continue with this example in this post.

With Eclemma, right-click on your unit test as shown before in our previous post on unit tests, but this time choose the “Coverage as” option.

After the unit test runs properly you will see these results:

![Full coverage](../../assets/test-coverage/full-coverage.png)

As you can see, for the BasketWeightCalculator class we reach 100% coverage. If we select only one test and run the coverage again we get a different percentage:

![Half coverage](../../assets/test-coverage/half-coverage.png)

The image shows that we are missing the “if” statement in the code, the red color indicating that tests on this part were not run at all. The part in yellow is for code that was run and tested for one condition, but not for the other.

## Determining the right amount of code coverage

Getting a 100% code coverage on the whole application is a bit unrealistic. A more realistic example is 60%-70%, assuming that this is your business logic code; however, depending on the kind of application, even that high of a % might be too time consuming.

I tend to follow [the Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle) as a good rule of thumb (the 80-20 rule) so that I can be sure to get code coverage of at least 20%. This 20% should be the “heart” of your application, e.g. code that changes often, breaks often and is a dependency to most other system components.

I can hear some of the more senior developers out there saying, “Aren’t you going to mention [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) as a metric for understanding how to write easier tests?” But this extends into other camps as well, so I’d prefer to cover that in another post on another day.

## Learn which tools rock!

As an additional section, I wanted to share with you a few advanced unit testing techniques that I’ve relied on a lot in the past. In our previous post on unit testing, we introduced JUnit and showed its basic capabilities.

JUnit luckily has several helper annotations that can cut down a lot on the amount of code you have to write. (N.B. if your application still uses JUnit 3.x, upgrade to JUnit 4.x NOW. The new features will save you a lot of time in the long run.)

Assume that you have already implemented a class that checks the validity for URLs called MyUriValidator. The class needs several statements to set up correctly. So your first attempt for a  unit test might be:

```java
public class MyUriValidatorTest {

  @Test
  public void test1() {
    MyUriValidator myValidator = new MyUriValidator();
    myValidator.allowFileUrls(true);
    myValidator.allowInternationlizedDomains(false);
    myValidator.allowReservedDomains(false);
    myValidator.allowCustomPorts(true);

    assertTrue("Domain is valid",myValidator.isValidUrl("http://www.google.com"));
  }

  @Test
  public void test2() {
    MyUriValidator myValidator = new MyUriValidator();
    myValidator.allowFileUrls(true);
    myValidator.allowInternationlizedDomains(false);
    myValidator.allowReservedDomains(false);
    myValidator.allowCustomPorts(true);

    assertTrue("Domain is valid",myValidator.isValidUrl("file://home/users"));
  }

  @Test
  public void test3() {
    MyUriValidator myValidator = new MyUriValidator();
    myValidator.allowFileUrls(true);
    myValidator.allowInternationlizedDomains(false);
    myValidator.allowReservedDomains(false);
    myValidator.allowCustomPorts(true);

    assertFalse("Domain is invalid",myValidator.isValidUrl("http://localhost:8080/"));
  }
}
```

There is clearly a lot of code of duplication here. JUnit has a [@Before annotation](https://junit.org/junit4/javadoc/4.12/org/junit/Before.html) for code that runs automatically before each test method. So your test can be simplified to:

```java
public class MyUriValidatorTest {

  private MyUriValidator myValidator = null;

  @Before
  public void beforeEachTest() { //Name of method does not actually matter
    myValidator = new MyUriValidator();
    myValidator.allowFileUrls(true);
    myValidator.allowInternationlizedDomains(false);
    myValidator.allowReservedDomains(false);
    myValidator.allowCustomPorts(true);
  }

  @Test
  public void test1() {
    assertTrue("Domain is valid",
        myValidator.isValidUrl("http://www.google.com"));
  }

  @Test
  public void test2() {
    assertTrue("Domain is valid",
        myValidator.isValidUrl("file://home/users"));
  }

  @Test
  public void test3() {
    assertFalse("Domain is invalid",
        myValidator.isValidUrl("http://localhost:8080/"));
  }

}
```

There is also the respective [@After annotation](https://junit.org/junit4/javadoc/4.12/org/junit/After.html) (runs after each test), as well as [@BeforeClass](https://junit.org/junit4/javadoc/4.12/org/junit/BeforeClass.html) and [@AfterClass annotations](https://junit.org/junit4/javadoc/4.12/org/junit/AfterClass.html) for code that runs ONCE before/after all tests.

So now we have an improved version. But even so, we have to create a new method each time a new URL needs to be tested. JUnit [supports parameterised tests](https://docs.junit.org/current/user-guide/#writing-tests-parameterized-tests) where you write a general test method once and then there is a separate method that provides the data.

Here is this approach for that:

```java
@RunWith(Parameterized.class)
public class MyUriValidatorTest {

  private MyUriValidator myValidator = null;
  private String uriTestedNow =null;
  private boolean expectedResult = false;

  public MyUriValidatorTest(String uriTestedNow,boolean expectedResult)
  {
    this.uriTestedNow = uriTestedNow;
    this.expectedResult = expectedResult;
  }

  @Parameters
  public static Collection data() {
    /* First element is the URI, second is the expected result */
     List uriToBeTested =  Arrays.asList(new Object[][] {
                        {  "http://www.google.com", true }, 
                          { "file://home/users", true }, 
                        { "http://staging:8080/sample", true },
                          {"http://localhost:8080/", false }  });

     return uriToBeTested;
  }

  @Before
  public void beforeEachTest() {
    myValidator = new MyUriValidator();
    myValidator.allowFileUrls(true);
    myValidator.allowInternationlizedDomains(false);
    myValidator.allowReservedDomains(false);
    myValidator.allowCustomPorts(true);
  }

  @Test
  public void testCurrentUri() {
    assertEquals("Testing for "+uriTestedNow, expectedResult,myValidator.isValidUrl(uriTestedNow));
  }

}
```

As you can see, adding a new URL is a single line change in the the method annotated as Parameters. You should examine all the capabilities of JUnit as contained in the WIKI.
Be sure not to miss [Exception Testing](https://github.com/junit-team/junit4/wiki/Exception-testing), [Theories](https://github.com/junit-team/junit4/wiki/Theories), [Rules](https://github.com/junit-team/junit4/wiki/Rules) and [Categories](https://github.com/junit-team/junit4/wiki/Categories).

For even more advanced capabilities check out [TestNG](https://testng.org/) which is a lot more advanced than JUnit. In fact JUnit is playing catch-up and some features of the latest versions are stolen/inspired (take your pick) from TestNG.

## Closing remarks

In this post we offered you some hints on which code you should test and which you should ignore. We also introduced the notion of code coverage and gave our recommendation for how much of the total code you should deal with. From the final example you should also see that investing some time in learning how to properly utilize unit testing features with JUnit, or if you feel more adventurous, TestNG, is highly recommended.

In future posts, we will look at how to write testable code (so that the unit tests you prepare to cover it are easier to write), the concept of mocking and when we need to use it.