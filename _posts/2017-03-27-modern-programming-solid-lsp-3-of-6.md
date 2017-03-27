---
layout: post
title:  "Modern Programming: SO[L]ID (3 of 6)"
date:   2017-03-27 15:40:00 -0700
categories: modern-programming solid lsp engineering-excellence
---
Modern Programming: SO[L]ID

The preceding chapter (Part 2) of this series can be found [here](https://bradleygibson.github.io/modern-programming/solid/ocp/engineering-excellence/2017/03/27/modern-programming-solid-ocp-2-of-6.html).

# Rationale
In this installment we'll look at how we can design software so that we can ensure that the parts (classes) are interchangeable without side-effects.  This is a fairly advanced topic.  The [TL;DR](http://gifhop.tumblr.com/post/52334669190) is 1) be "consistent" in your class definitions; design classes with 2) loose coupling and 3) high cohesion.

Traditional, legacy codebases are tightly coupled, with relatively low cohesion--precisely the opposite of what LSP prescribes.  This fact may be the *single largest contributor* to unexpected bugs and regressions when making changes in our code.

# L - Liskov Substitution Principle (LSP), part 1
In its simplest form, LSP states that derived classes must not change behavior established by the base class in any way.

How can we know we have achieved this?

A software entity (class, method, function, type, module, etc.) which operates on an object must be able to use objects derived from that object without knowing it.  When this condition is true, it can be said that the derived object is a *behavioral subtype* of its base object.

To be a behavioral subtype of its base, an object or class must meet six conditions (my apologies in advance--did I mention that this was a fairly advanced topic?  Even if you skim the following definitions, the practical discussion of how they're applied is in part 2 of the LSP discussion [here]()):
1) **Contravariance of parameters**: The parameters provided to methods in the subclass must be at least as permissive as their base class counterparts.  They can be more permissive than the base counterparts.

2) **Covariance of return types**: The return values emitted from methods in the subclass must be at least as restrictive as their base class counterparts.  They can be more restrictive than the base counterparts.

3) **Pre-conditions & Post-conditions**: Pre-conditions and post-conditions are conditions that must be true before or after (respectively) a given method on a class is called.  For instance, a database connection has been opened (pre-condition) or written file has been closed upon completion (post-condition). Pre-conditions for the subclass must be no more restrictive than the base class.  Post-conditions for the subclass must be no more permissive than the base class.

4) **Invariants**: Invariants are relevant conditions that do not change.  After a method has returned, any conditions relevant to the method are left unchanged from how they were before the method was called.  For example, a method which opens, uses and closes a database connection might require the connection to be in the closed state before it is called.  Because the connection is relevant to the method, and it is closed both before and after the method call, it is an invariant.  To comply with LSP, a base class’ invariants must not be changed by a subclass.
 
5) **History Constraint**: Subclasses must not allow mutation of any state that exists in the base class in any way that is not permitted by the base class itself.  For example, it would be a violation of LSP for a subclass to make a base class immutable property mutable in a subclass.

6) **Exceptions**: Subclasses may only throw exceptions that are of the same type as the base class or are derived from one of those types.

Ok, so there is this set of somewhat dry, academic “contracts” we must meet to ensure our derived class does not change behavior established by the base class.  Informally, these essentially amount to **"don't let your derived classes accept values, return values, or behave in ways that the parent class is not permitted to."**

But why should we bother with LSP at all?  What advantage(s) does LSP compliance provide?

Ultimately, the benefit boils down to this: **any LSP-violating software entity will have intimate knowledge of the inner workings of a type it depends on.**  Even worse, any deviations in behavior introduced by descendants of the type it depends on will also need to be hard-coded into the software entity as well.  While this situation is a violation of many SOLID principles, from LSP’s perspective, **we have created a undesirable situation known as *tight coupling***.

Coupling is an indication of how well or poorly insulated one software entity is from changes in another.  If one entity depends on (what should be) internal implementation details of another to function properly, then it is highly coupled, and is more likely to break when the other is changed.  It is easy to see that this makes code more expensive and difficult to maintain, as changes in one entity break others, necessitating their change, breaking still others, and so on. 

It is worth pointing out that some non-trivial portion of the code in the LSP-violating software entity is now *dedicated to handling internal, special-case details of its dependency and possibly subclasses too*.  The entity is **no longer strictly focused on solving a crisp, specific business problem**, because it now has significant logic dedicated to managing and “wrangling” the desired behavior from its various dependencies.

Put more formally, another undesirable characteristic, *low cohesion* frequently results from tight coupling.  We see this as a lot of "glue" code which magically keeps things working.  Mostly.

Think of cohesion as a measure of how focused a software entity is to a specific job or task.  If an entity does a great many different things, it has low cohesion--with the ultimate expression of this being a god object.

In the next installment, we’ll look at a practical example of code without and with LSP applied.
