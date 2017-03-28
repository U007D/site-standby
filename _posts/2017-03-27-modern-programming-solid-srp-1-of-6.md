---
layout: post
title:  "Modern Programming: [S]OLID (1 of 6)"
date:   2017-03-27 14:49:00 -0700
categories: modern-programming solid srp engineering-excellence
---
# Rationale
The primary motivation of these articles is to reduce the cost of maintenance of code.  This cost typically dwarfs the cost of writing code, since code is written "once" and then maintained for a (hopefully) long period of time ([article #5](https://bradleygibson.github.io/modern-programming/solid/srp/engineering-excellence/2017/03/27/modern-programming-solid-isp-5-of-6.html) provides research data to back this).

While everyone understands the process of bug-fixing, when we consider the addition of new features into the codebase as a form of maintenance (changing some specific behavior, while ensuring we have not broken or regressed existing behavior), it becomes easy to see why optimizing code for maintainability is so essential.

# Modern Programming: SOLID
If you have been writing software for any period of time, you have witnessed the rise and fall of various languages, programming paradigms and philosophies.
 
I am continually looking at those ideas which have proven themselves and am attempting to incorporate several industry best-practice concepts into my approach to software engineering.  I refer to the collective representing the ideas we choose to put into practice as “Modern Programming”.  (Adapted from the term “Modern C++”, coined by Andrei Alexandrescu, used today to describe a new and changed collection of important C++ styles and idioms, debuting in C++11).
 
SOLID principles are an important aspect of Modern Programming
 
SOLID is itself a “package” of five ideas (originally referred to as the first five principles of object-oriented programming, until Robert Martin coined the mnemonic acronym).  A key goal of the SOLID approach is to address traditional difficulties efficiently and reliably maintaining and extending existing source code.
 
In so doing, it is fair to say that there is an up-front cost to writing SOLID code--namely, an investment in a typical software engineering skillset.  This up-front investment realizes a return in the form of code that has a longer useful lifespan, since it results in a more maintainable and a more extensible code base.  Further, SOLID code will be broken into far more "little pieces" than ["conventionally" written code](http://williamdurand.fr/2013/07/30/from-stupid-to-solid-code/).  Many developers will consider these smaller pieces to amount to an unmaintainable mess, but the best advice I can provide is to reserve judgment until after having lived with it through a full major release cycle.  It will pay clear dividends, and the concern around the number of small components will wane with experience.

While all of these concepts are language-agnostic, I'll sometimes use C# as C++ 'pseudocode' for clarity, enabling better focus on the concepts being discussed.

So let’s jump in and look at some specifics.
# S - Single Responsibility Principle (SRP)
The Single Responsibility Principle states that a class should have exactly one responsibility or job to do, from the perspective of the software’s specification.  Take, as an arbitrary, simple example, a data store, whose interface might look something like this (C#):
 
{%highlight csharp linenos%}
public interface IDataStore<T>
{
    void Add(T item);       //*See also Fluent API note below.
    IDataStore<T> Save();
}
{%endhighlight%}
 *Consider `IDataStore<T> Add(T item)` instead for a [Fluent API design](https://en.wikipedia.org/wiki/Fluent_interface) (more about these in another article).
 
A typical implementation might be:
        
        public class DataStore<T> where T : class
        {
            private IPersistentStore store;  //Some interface to persistent storage
 
            public DataStore(IPersistentStore store)
            {
                this.store = store;
            }
 
            public void Add(T item)
            {
                this.store.Add(T item);
            }
 
            public IDataStore<T> Save()
            {
                try
                {
                    this.store.Persist();
                }
                catch (Exception e)
                {
                    File.AppendAllText(pathToLog + "log.txt", e.ToString());
                    throw;
                }
                return this;
            }
        }
 
The above `DataStore` class abstracts the backing store and will log an event if there is a problem writing to that backing store.  That seems both reasonable and straightforward.

What SRP says about this implementation is that it is responsible for both delivering T to the backing store and logging of failures.  Of course, failures must be detected, but SRP advises that how they are handled should be out of scope for the DataStore class.
 
Why go through the extra trouble?  Consider a failure when *writing to the log*.  If a log failure is considered critical, then an alternative method of alerting the right people about the log failure must also be added to this class (ie. more logic).  This logic will be required *everywhere* a log is written, so the natural way to encapsulate all the logging requirements into a single, consistent source of truth (ie. make it [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)) is to have a separate Logger class which anyone with logging needs will take a dependency on.  Should the application’s logging needs evolve, there is a single place to go to upgrade capability consistently throughout the app.

The `DataStore` class should then depend on having an object which implements an `ILogger` interface before `DataStore` can be properly instantiatied.  When `DataStore` needs to log, it will call whatever concrete `Ilogger` instance it has at runtime via a stable, compiler-enforced API.  See article #6 (Dependency Inversion Principle) for more on ways to manage these dependent relationships. 
 
Ultimately, this is how SRP yields lower maintenance costs and more reliable extensibility when it comes to the issue of code maintenance.  I hope this simple, but real-world-applicable example has shown how separating the concerns of the `Logger` from the `DataStore` yields more stable, maintainable code--specifically, code that resists failing unexpectedly in one place code base as a result of another part of the code base being upgraded.
