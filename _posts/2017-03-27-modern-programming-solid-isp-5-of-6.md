---
layout: post
title:  "Modern Programming: SOL[I]D (5 of 6)"
date:   2017-03-27 16:47:00 -0700
categories: modern-programming solid srp engineering-excellence
---
I - Interface Segregation Principle (ISP)

The preceding chapter (Part 4) of this series can be found [here](https://bradleygibson.github.io/modern-programming/solid/lsp/engineering-excellence/2017/03/27/modern-programming-solid-lsp-4-of-6.html).

ISP is very straightforward. It says that a client of an interface should not be forced to depend on methods it does not use (in other words, don’t create “fat interfaces” or “god objects”). The target interface should be broken down into separate, simpler interfaces, each with an eye on delivering only related functionality to its respective clients.

Adhering to this approach will tend to yield more, smaller interfaces than a non-ISP-compliant implementation. But having more, tiny pieces of anything is a form of complexity, so how can this be a good thing?

The benefit is that clients no longer need to be aware of the APIs they are not using. Also, notice how ISP compliance more naturally lends itself to SRP and LSP, other SOLID principles. In short, while it's fair to suggest ISP makes the program harder to write, as a reward, it also delivers a proportionally larger (measurable) benefit of an easier-to-use and easier-to-maintain solution (C#).

Let’s look at an example in code:

    //ISP-violating example
    public class Employee
    {
        public string Name { get; set; }
        public string Title { get; set; }
        public PhoneNumber Phone { get; set; }
        public EmailAddress Email { get; set; }
        //...
    }

    public class Caller
    {
        CallResult Call(Employee employee)
        {
            //Call employee
        }
    }
    
    public class Emailer
    {
        bool Email(Employee employee, string subject, string body)
        {
            //Email employee
        }
    }

At first glance, this might seem like a reasonable (and very common) separation of concerns. ISP asks us to look at the burden we are imposing on `Employee`’s clients. Should `Caller` have to know what to do with (or not to do with) `Email`? Should `Emailer` need or want to know about `Phone`?

Let’s refactor this with ISP in mind:

    public interface INameable
    {
        string Name { get; set; }
    }
    
    public interface ICallable : INameable
    {
        PhoneNumber Phone { get; set; }
    }
    
    public interface IEmailable : INameable
    {
        EmailAddress Email { get; set; }
    }
    
    public class Employee : INameable, ICallable, IEmailable
    {
        public string Name { get; set; }
        public string Title { get; set; }
        public PhoneNumber Phone { get; set; }
        public EmailAddress Email { get; set; }
        ...
    }
    
    public class Caller : ICallable
    {
        CallResult Call(ICallable person)
        {
            //Call person (Only Name and Phone are available)
        }
    }
    
    public class Emailer : IEmailable
    {
        bool Email(IEmailable person, string subject, string body)
        {
            //Email person (Only Name and Email are available)
        }
    }

Now `Caller` deals with a focused and consistent interface, as does `Emailer`. This code is easier to update or extend with fewer “surprises” as a result of this focus. As the problem gets more complex, the value of ISP *increases*--consider a product inventory interface (and the variations in interface required by different types of product) or security interfaces (where ideally, some API’s would not even be available to some types of user). In such a system, with potentially dozens of product- or domain-specific API endpoints--having every client understand which are relevant to its specific task and which not to touch when “everything” is exposed becomes a significant and completely unnecessary burden for client implementation and maintenance.

ISP recommends (typically) making a larger number of simpler interfaces. It asks us to think through what is expected to be needed where (ie. to design the API); this may be significant work.

We saw even in the simple code sample above, in becoming ISP-compliant, the code expanded from 24 lines of code to 39. Usually, as software engineers, we are not excited about increasing the number of lines of code--that is often opposite to the direction we are trying to go. So it’s reasonable to ask, with the increased requirement for planning, the extra classes and ultimately additional code, aren’t we making writing code harder? More expensive? Perhaps even (*shudder*) take longer to write??

In a word? Yes.

Here is the argument on why doing this makes good sense:

![alt text](https://github.com/bradleygibson/bradleygibson.github.io/blob/master/_posts/assets/quality-roi.png?raw=true "Rate breakdown of costs incurred developing poor and high-quality software.  Summary: Poor quality is cheaper until the end of the coding phase.  After that, high quality is cheaper.")


As you can see from the above graph, improving the quality of software is not free. In fact, poor quality software is, in fact, cheaper for the organization, but just until code complete. When you consider the length of time it takes to write code vs. the amount of time the code will live thereafter, it is clear that, at code complete, the overwhelming majority of the code’s lifetime lies ahead of it.

So is this true, in reality?  Let’s look at what the data tells us of how costs break down over the lifetime of a software project, from cradle to grave:

![alt text](https://github.com/bradleygibson/bradleygibson.github.io/blob/master/_posts/assets/software-tco.gif?raw=true "Pie chart of Software Development Lifecycle costs.  Summary: Coding: 12%, Maintenance 67%")
Source: Schach, R. (1999), Software Engineering, Fourth Edition, McGraw-Hill, Boston, MA, pp. 11.

The above graph illustrates why it is so worthwhile to adopt engineering disciplines like SOLID to develop more extensible, maintainable code. The lion’s share of costs for a software project come as maintenance costs, and so investments here have the potential to provide more than 5x the return of the next most costly category (coding).  SOLID aims to streamline maintenance by yielding a codebase with fewer bugs, greater flexibility and a longer useful life. By taking a bite out of maintenance costs, a SOLID project will both shrink the cost pie and reduce the proportion of the cost attributable to maintenance.

(Note: this analysis externalizes (does not include) any infrastructure or IT costs associated with deploying and maintaining a software deployment)--factors that make the maintenance stage of software development even costlier.

While it is always possible to overdo things by blindly following principles, with practice, teams can find the right balance to keep development costs manageable while addressing the problem of the overwhelming cost of maintenance.
