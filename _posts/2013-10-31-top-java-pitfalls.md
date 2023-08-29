---
layout: post
title: Top 10 Java pitfalls of experienced Java developers
category: java
---

_This post was originally published at the [JRebel/Zeroturnaround blog](https://www.jrebel.com/). Reposting here since it is not available there any more._

## Introduction

Can we start by asking a serious question? How easy is it to find advice for novice Java programmers on the web? Whenever I look, I see tons of tutorials, books and resources for a programmer who is just starting out. At the same time, there is a ton of information on how to get a wide perspective on a large enterprise project: scaling your architecture, message busses, database interconnections, UML diagrams and other high-level stuff is well documented.

But what about those of us who are just experienced, professional Java developers? This area is lacking coverage, and programming advice for the senior developer, team leader or the junior architect is hard to find. Numerous resources focus only on one end of the spectrum, either dealing with extreme code details or talking about general architectural concepts. This travesty must come to an end!

Now, I’m all about good practices and thoughtful code, and I think that there is plenty of good general advice to give to senior developers at the intermediate level. I wrote down 10 of the most common pitfalls I see committed by experienced programmers in my professional life; most of the time, these things happen because of misinformation and lack of attention, which are easy things to remedy.

So let’s begin the countdown of pitfalls / dumb moves that you might make if you aren’t paying close attention! I’ve placed these in descending order based on annoyance – #1 is what bothers me most, but all 10 can hurt you equally depending on additional factors ;-)

## Common Pitfall #10: Misusing or misunderstanding dependency injection

[Dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) is generally considered a “good” thing to have in an enterprise project. You can’t go wrong if you use it. But is this actually true?

One of the basic ideas of DI is that instead of having an object looking for its dependencies, you initialize the dependencies before creating the objects in a well defined manner (using a DI framework) and when the object is actually created you just pass it the preconfigured objects in the constructor (constructor-injection) or using a method (method injection).

However the whole point is that you pass what the object needs and nothing more. I still however find code like this even in brand new projects:

```java
public class CustomerBill {
        	
    //Injected by the DI framework
    private ServerContext serverContext;
        	
        	
    public CustomerBill(ServerContext serverContext)
    {
        this.serverContext = serverContext;
    }
        	
    public void chargeCustomer(Customer customer)
    {
        CreditCardProcessor creditCardProcessor = serverContext.getServiceLocator().getCreditCardProcessor();
        Discount discount  = serverContext.getServiceLocator().getActiveDiscounts().findDiscountFor(customer);
                    	
        creditCardProcessor.bill(customer,discount);
    }
}
```

This is not true DI. The object still does initialization on its own. The presence of “train code” in line `serverContext.getServiceLocator().getCreditCardProcessor()` is another indication that something is wrong here.

Of course, the correct way would be to pass the final objects directly like this:

```java
public class CustomerBillCorrected {
        	
    //Injected by the DI framework
    private ActiveDiscounts activeDiscounts;
        	
    //Injected by the DI framework
    private CreditCardProcessor creditCardProcessor;
        	
        	
    public CustomerBillCorrected(ActiveDiscounts activeDiscounts,CreditCardProcessor creditCardProcessor)
    {
        this.activeDiscounts = activeDiscounts;
        this.creditCardProcessor = creditCardProcessor;
    }
        	
    public void chargeCustomer(Customer customer)
    {
        Discount discount  = activeDiscounts.findDiscountFor(customer);
                    	
        creditCardProcessor.bill(customer,discount);
    }
}
```

## Common Pitfall #9: Pretending Java is more like Perl

One of the nice properties of Java compared to other languages is its approach to type safety. On small projects where you are the only developer, you can pretty much get away with any coding style you like. However, on big Java code bases and complex systems, you need early warnings when something goes wrong. Most programming errors should be caught on compile time and not on runtime.

Java gives a lot of facilities in order to facilitate these compile time warnings. But there is not much that it can do when you shoot yourself in the foot writing code like this:

```java
public class AnimalFactory {
        	
        public static Animal createAnimal(String type)
        {
            switch(type)
            {
                case "cat":
                    return new Cat();
                case "cow":
                    return new Cow();
                case "dog":
                    return new Dog();
                default:
                    return new Cat();
            }
        }
 
}
```

This code is very dangerous since no compile time check can save you. A developer might call your factory using a misspelled string like `createAnimal(“dig”)` expecting a dog (a not a cat). The code will compile fine and the error will only appear later during runtime. Depending on the application the appearance of the error might be after one month when the application has reached production! Ouch.

Do yourself a favour and use all facilities Java offers for compile time safety. Here is a more correct approach (there are other possible solutions) that only compiles when it is correct.

```java
public class AnimalFactoryCorrected {
    public enum AnimalType { DOG, CAT,COW,ANY};
        	
    public static Animal createAnimal(AnimalType type)
    {
        switch(type)
        {
            case CAT:
                return new Cat();
        	case COW:
                return new Cow();
            case DOG:
                return new Dog();
            case ANY:
            default:
                return new Cat();
        }
    } 
}
```

## Common Pitfall #8: Pretending Java is more like C (i.e. not understanding OOP)

Back in the days of C, the suggested approach for writing code was the [procedural way](http://en.wikipedia.org/wiki/Procedural_programming). Your data exists in [structs](http://en.wikipedia.org/wiki/Struct_(C_programming_language)) and operations happen on data via functions. The data is stupid and the methods are smart.

Java is however [an object-oriented language](http://en.wikipedia.org/wiki/Object-oriented_programming), the reverse of this approach. Data and functions are bound together (creating classes) that should be smart on their own.

A lot of Java developers, however, either do not understand the difference or they never bother to write OOP code, even though deep down they know that their procedural approach seems out of place.

One of the best indicators of procedural code in a Java application is the usage of the `instanceof` operation and the respective upcasting/downcasting code segments that follow it. The `instanceof` operator has its valid uses of course, but in the usual enterprise code it is a [huge anti-pattern](http://www.javapractices.com/topic/TopicAction.do?Id=31).

Here is an example that does not deal with animals:

```java
public void bill(Customer customer, Amount amount) {

    Discount discount = null;
                    
    if(customer instanceof VipCustomer)
    {
        VipCustomer vip = (VipCustomer)customer;
        discount = vip.getVipDiscount();
    }
    else if(customer instanceof BonusCustomer)
    {
        BonusCustomer vip = (BonusCustomer)customer;
        discount = vip.getBonusDiscount();
    }
    else if(customer instanceof LoyalCustomer)
    {
        LoyalCustomer vip = (LoyalCustomer)customer;
        discount = vip.getLoyalDiscount();
    }

    paymentGateway.charge(customer, amount);

}
```

This code can be [refactored in an OOP way](https://www.artima.com/interfacedesign/PreferPoly.html) like this:

```java
public void bill(Customer customer, Amount amount) {

    Discount discount = customer.getAppropriateDiscount();

    paymentGateway.charge(customer, amount);

}

```

Each class that extends `Customer` (or implements the `Customer` interface) defines a single method for the discount. The beauty of this is that you can add new types of `Customer` without touching the customer management system. With the `instanceof` variation, adding a new customer would mean that you would have to search the customer printing code, the customer billing code, the customer contact code etc. and add a new if statement for the new type.

You should also check out this discussion on [rich versus anemic domain](https://en.wikipedia.org/wiki/Anemic_domain_model) models.

## Common Pitfall #7: Using excessive lazy loading (i.e. not understanding your object lifecycle)

More frequently that I’d like, I discover code like this:

```java
public class CreditCardProcessor {
    private PaymentGateway paymentGateway = null;

    public void bill(Customer customer, Amount amount) {

    	//Billing a customer always needs a payment gateway anyway
      	getPaymentGateway().charge(customer.getCreditCard(), amount);

    }

    private PaymentGateway getPaymentGateway()
    {
        if(paymentGateway == null)
        {
            paymentGateway = new PaymentGateway();
            paymentGateway.init(); //Network side Effects  here
        }
        return paymentGateway;
    }

}

```
The original idea behind lazy loading is valid: If you have an expensive object, it makes sense to create it only if is needed. However, before applying this technique you have to be really sure that:

 * The object is really “expensive” (how do you define that?)
 * There are cases when the object is not used (and thus it does not need to be created)

I increasingly see this `if` structure in objects, which either are not really “heavy” or objects that get created always during runtime – so what is the gain?

The main problem of overusing this technique is the fact that it hides the lifecycle of your components. A well-built application has a clear lifecycle of its main constructs. It should be clear when objects are created, used and destroyed. Several DI frameworks can help you with your object lifecycle.

But the truly hideous use of this technique comes from into the picture when object creation has side effects. This means that the state of your application depends on the order of object creation (the order of the type of requests that come in). Suddenly debugging your application is next to impossible since there are so many cases to cover. Replicating an issue that happened in production is a huge task because you have to know the order in which the if statements run.

Instead of using this, just define all the objects needed during application startup. This has also the added advantage that you can catch any fatal issues when they appear during application deployment.

## Common Pitfall #6: Depending on the ‘Gang Of Four’ (GOF) book as your bible (a.k.a. the GOF religion)

I truly envy the writers of the [Design Patterns book](https://en.wikipedia.org/wiki/Design_Patterns). This single publication has affected the whole programming industry in a manner that no other book could possibly trump. Design patterns have even entered the job interview process along the way and it seems like sometimes you cannot land an IT position if you didn’t read the book and memorized the names and formulations of several design patterns. Hopefully that era is slowly dying.

Now don’t get me wrong; the book is not bad by itself. Like so often throughout history, the problem stems from how people use and interpret it. Here is the usual scenario:

1. Mark, the architect, gets his hands on the GOF book and reads it. He thinks it’s darn cool!
1. Mark looks at the current code base he is working on.
1. Mark selects a design pattern he likes and applies it in the code he works at that point in time
1. Mark passes along the book to senior developers who start the same cycle from step 1.

The resulting code is a mess.

The correct way to use the book is *clearly described in its introduction* (for those who actually bother to read it). You have a problem that you keep stumbling over and over again and the book provides you with solutions that have worked with similar problems in the past. 

Notice the correct sequence of events. I have a problem, I look at the book and **then** I find a solution to my problem.

Don’t fall into the trap of looking at the book, finding a solution you like and then attempting to apply it in your code in some random place, especially since some of the patterns mentioned in the book are no longer valid (see #5 below).

## Common Pitfall #5: Using Singletons (it’s an anti-pattern!)

I already mentioned design patterns in my previous point. [Singletons](http://en.wikipedia.org/wiki/Singleton_pattern) however need a point on their own. I’ll ask you all to repeat after me 100 times ;-)

* A singleton is an anti-pattern
* A singleton is an anti-pattern
* A singleton is an anti-pattern
* A singleton is an anti-pattern
* A singleton is an anti-pattern…

Singletons had their use at one point in time. But with modern dependency injection frameworks, singletons can be completely eliminated. When you use a singleton, you are really introducing just a new set of problems in your code. Why? Because:

1. A singleton creates hidden dependencies in your class
1. A singleton makes code untestable (even with mocking)
1. A singleton mixes resource creation with resource acquisition
1. A singleton allows for side effects on global state
1. A singleton is a possible source of concurrency problems

There is plenty of documentation on why a singleton is now an anti-pattern if you still don’t believe me. Search and you shall find it.

## Common Pitfall #4: Ignoring method visibility

I am always amazed when I meet experienced Java developers who think that Java has only three protection modifiers. Well it has four(!), because there is also **package private** (a.k.a. **default**). And no, I am not going to explain here what it does. Go look it up.

What I am going to explain is that you should pay attention to what methods you make **public**. The public methods in an application are the application’s visible API. This should be as small and compact as possible, especially if you are writing a reusable library (see also the [SOLID principles](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)).

I am really tired of seeing public methods that should be **private**. Not only because they expose the internal implementation details of class, but also because almost never should they be used outside of this class in the first place.

A corollary to this is that you always use unit tests for the public methods of your class. I have seen so-called architects who believe that that it’s acceptable to convert private methods to public so that they can run unit tests against them.

[Testing private methods is simply wrong](https://shoulditestprivatemethods.com/). Just test the public method that calls the private one. 

Remember: in no case you should expand your API with more public methods only because of unit tests.

## Common Pitfall #3: Suffering with project-specific StringUtils (i.e. or the more general NIH syndrome)

It used to be in the old days that every sufficiently large Java project contained files like `StringUtils`, `DateUtils`, `FileUtils` and so on. Now, I understand this when I see it in legacy code. And believe me, I feel your pain! I know of all the effort that has gone into these files to make them mature and stable since so much code depends on them.

But when I see files like that on brand-new code, with no dependencies, a part of me cringes inside. One of the responsibilities of architects and senior developers is to keep track of well-tested existing solutions that are much easier to integrate in your application.

If your job title contains the word architect in it, there is no excuse for not knowing [Apache Commons](http://commons.apache.org/), [Guava libraries](https://github.com/google/guava) or the [Joda Date library](https://www.joda.org/joda-time/).

You should also do some research before writing code on an area with which you are not familiar. For example, before writing a REST service that creates PDF files, spend some time to understand what REST frameworks exist for Java and what is the suggested method for creating a PDF file. (Running a unix command line app is **NOT** the suggested method. Java can create PDF files on its own via [itext](https://itextpdf.com/) and [PDFBox](https://pdfbox.apache.org/)).

## Common Pitfall #2: Environment-dependent builds

I’ve been working with Java development teams for 10 years now, and let me tell you a secret: **there is only one acceptable way to build an enterprise application**. Here’s how you do that:

1. I show up one day as a newcomer in your software company.
1. I get oriented on my workstation and install Java and my favourite IDE/tools.
1. I check out the code from the company repository.
1. I spend 5 minutes at most to understand what build system you have (usually Maven or Gradle).
1. I run **a single command** to build the application and it succeeds.

This is not just the optimal scenario, but the only valid one if you have paid attention to how the application builds.

If your enterprise application is dependent on a specific IDE, specific versions of IDE plugins, local files on the PC, extra, undocumented environment settings, network resources or anything non-standard, then it is time to rethink your build system.

Also related is the fact that a build should be performed in one step (See [the Joel Test](https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/)).

Now don’t get me wrong–the integration tests, a self-check build or extra deployment/documentation steps can have extra settings or use external databases (that should be documented in your company wiki).

But the simple compilation scenario to obtain the executable should be a matter of an hour at most. I have seen projects where the first compile for a new hire takes 2 days (in order to setup the environment).

## Common Pitfall #1: Using Reflection/Introspection

News flash: If you are writing an [ORM framework](http://en.wikipedia.org/wiki/Object-relational_mapping), a Java agent, a meta-compiler, an IDE or something exotic then you could probably use Java reflection if you wish. However, for most enterprise applications, which really are just a glorified [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) interface over a data store, [Java reflection](https://docs.oracle.com/javase/tutorial/reflect/) is an overkill.

I have often seen reflection used in the name of performance, backwards compatibility or even forwards compatibility. Almost always it is used wrong. Reflection is always based on several assumptions (e.g. method name conventions) that might be true today but not tomorrow.

Reflection issues are very difficult to understand, very difficult to debug and very difficult to fix.

And I am not even going to delve into cases of [self-modified code](http://en.wikipedia.org/wiki/Self-modifying_code). Using reflection in an enterprise Java application is like inserting a time bomb in the foundations of a building. The building might be stable now, but as soon as the timer kicks off, everything falls into pieces.

What always puzzles me is the fact that reflection is always seen as “essential” by the architect who introduces it, while in reality you can solve the same issues more cleanly with a carefully constructed object hierarchy and the clear definition/documentation of a plugin’s architecture.

So let me spell this out for all you “architects” out there. It **IS** possible to create a stable, fast, and maintainable enterprise Java application **WITHOUT** using reflection. If your vanilla-flavoured enterprise app uses reflection then you are just asking for trouble.

## Bonus section: Only try optimization when you have knowledge of the actual bottleneck

I have seen architects who spend great effort on:

 * Fine-tuning logging statements
 * Replacing Vectors in legacy code
 * Replacing `+` operators with `StringBuffers` in loops
 * Refactoring existing, stable, mature and bug-free code for the sake of “performance”
 *   …and other fancy stuff

Sadly, this is done without actually measuring what effect these efforts might have. Even if the application runs a bit faster, most of the times there are other more serious problems (like database locking, memory leaks, streams that are never closed/flushed) that nobody pays attention to.

Spending time to fix logging and gain 3% in speed is not very helpful when fixing the SQL queries would give 200% gains in speed.

Therefore, it’s important to keep in mind that any change that happens for performance reasons is actually a four-step process:

1. Measure the current system (valid benchmarks are a big task on their own)
1. Apply the change
1. Measure again
1. Evaluate the effort given and the performance gained ratio.

Remember the mantra for optimizations – **measure, don’t guess**.

## Conclusion

If I seem a bit grouchy, it’s because I’ve had more than my fair share of cleaning up the messes left by experienced coders. In a way is even worse than fixing the mistakes made by beginners — senior developers should know better! ;-)

Hopefully, this list of 10 common pitfalls made by the senior developer helps you discover some areas where your expertise should be put to use in problem solving. Even the best coders out there should remember that there is always something to [re-]learn and put into practice.
