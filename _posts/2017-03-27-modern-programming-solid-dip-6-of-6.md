---
layout: post
title:  "Modern Programming: SOLI[D] (6 of 6)"
date:   2017-03-27 17:33:00 -0700
categories: modern-programming solid dip engineering-excellence
---
D - Dependency Inversion Principle (DIP)

The preceding chapter (Part 4) of this series can be found [here](https://bradleygibson.github.io/modern-programming/solid/srp/engineering-excellence/2017/03/27/modern-programming-solid-isp-5-of-6.html).

DIP is probably the least-well understood (and the most misunderstood) of the SOLID principles.

If you are unfamiliar with the concept of dependency injection, then you can safely ignore the rest of this paragraph for now.  For everyone else, it is important to call out that dependency inversion != dependency injection. While adhering to DIP may indeed lead you to employ some flavor of dependency injection (eg. Constructor, Service Locator, Inversion of Control or other pattern), it is important to call out that dependency inversion and injection are not synonymous. (The distinction is perhaps worthy of a future article.)  For now, understand that Dependency Injection is but one means of achieving DIP.

[Robert Martin (aka “Uncle Bob”)](https://en.wikipedia.org/wiki/Robert_Cecil_Martin) coined the SOLID mnemonic. He (somewhat academically) defines DIP as follows:

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

> Abstractions should not depend on details. Details should depend on abstractions.

Let’s parse this to make it easier to digest (and more importantly, at least speaking for myself, to retain).

Uncle Bob uses both the terms *“low-level modules”* and *“details”* interchangeably to refer to a dependency--think of an interface that some class needs in order to do its job.  And if that *dependency* is to depend on an interface (as the latter quote above says it should), typically, we would think of this "detail" a concrete class--instantiation or an implementation of an interface.

He uses the term *“abstractions”* to encompass both of what are commonly referred to as interfaces as well as abstract classes--interfaces which cannot be instantiated.

And finally, to “high-level modules".  These are units of code (classes, modules, etc.) that are concerned with behavior (think business rules), but not implementation details. Uncle Bob writes they are *“the truths that do not vary when the details are changed”*. For example, the purpose of a `Button` class might be to turn something else on or off. Should the `Button` class be concerned with what the something else is? Nope--that would violate several SOLID principles, including DIP. Should the `Button` class care whether it represents a physical, mechanical button, or a graphical button on a touch-screen? Nope--same reasons as above. So the `Button` class represents a high-level module, and what it turns on or off is a “low-level module” (an implementation) that it should not care about.

So, to simplify, in our world:

All implementations should depend on interfaces (or abstract classes).

Let's see if this clarification satisfies Uncle Bob’s original tenets:

> “High-level modules should not depend on low-level modules”.

Check.  No high-level concrete or abstract classes depend on low-level concrete classes.  Instead, these modules depend on interfaces (what he refers to as "abstractions").

> “[High-level and low-level modules] should depend on abstractions”.

Check. As per above, all dependencies will be created against interfaces.

> “Abstractions should not depend on details”.

Check. Since Interfaces do not contain any implementation, by definition, no dependencies can be created there. If either an interface or abstract class inherits from another class, or an abstract class’ implementation has a dependency, recall again that all dependencies will be created against interfaces.

> “Details should depend on abstractions”.

Check. All depend… Never mind--you get the idea. ;)

There is one subtle difference between my simplification above, and the original guideline. To cut to the chase, it is that the interfaces that are created are not *owned* by the low-level implementation. One can think of ownership in this case as the ability to compile: with a dependency on an interface, rather than a concrete class, a high-level module should be able to compile without a low-level implementation available.

Now this may seem odd, that an implementation (eg. Foo) does not “own” (is not packaged with) its own interface (eg. IFoo). The interface can be packaged with the high-level interface (ie. the implementation that depends on it) or all on its own, depending on the needs of the project, just as long as it’s not packaged with its own implementation. This important distinction makes our shorthand definition complete.

So to summarize in our own words, DIP recommends:

* All implementations should depend on interfaces (or abstract classes).

* Implementations should not own the interfaces they implement.  (Here, the term "own" means *"to be packaged with"* or *"drive changes into"*).
(More on what "drive changes into" means below).

Whew! Ok, so now we have a straightforward, comprehensible guideline for how to create dependencies. So what's the benefit of doing this?

Let’s look at an example in code to see it in practice, and perhaps discover why we might want to do DIP in the first place:

    //Non-DIP implementation
    public class Switch
    {
        private Lamp lamp = new Lamp();
        void On()
        {
            lamp.TurnOn();
        }
    }
    
    public class Lamp
    {
        public void TurnOn()
        {
            //Lamp implementation details
        }
    }

We can see that `Switch` clearly has a dependency on `Lamp`. If `Lamp` needs to change, it’s entirely possible that those changes will ripple through to its interface, which would mean potential changes to any and all classes that depend on it. By now, you have seen that SOLID principles are all about compartmentalizing change--ensuring that, as much as possible, change happens in specific, focused areas so that uncertainty caused by change is localized.

You’ll also recall that SOLID also promises a longer useful life for our code. Here is an example of how this is accomplished: In the above example, `Switch` is clearly not reusable--it can’t turn on a `Heater` or a `Motor` or a `VacuumCleaner` because it has a `Lamp` baked right in.

Let's make some changes:

    public interface ISwitchableDevice
    {
        void TurnOn();
    }
    
    public class Switch
    {
        private ISwitchableDevice device;
    
        Switch(ISwitchableDevice device) //Constructor dependency injection
        {
            this.device = device;
        }
    
        public void On()
        {
            device.TurnOn();
        }
    }
    
    public class Lamp : ISwitchableDevice
    {
        public void TurnOn()
        {
            //Lamp implementation details
        }
    }

Now `Switch` takes any object with implements the ISwitchableDevice interface in its constructor, and stores that instance in a private variable for later use.  This form of DIP is known as **Constructor [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection)**. For now, let's just agree that "something else" not shown here (eg. Factory, initialization routine, config file, etc.) decides what `ISwitchableDevice` should be created and passes an instance to `Switch`’s constructor. It’s not important for this article what the “something else” is (and it could be a whole set of blog posts unto itself).

Now any class willing to implement the `ISwitchableDevice` interface can be controlled by our `Switch`. It’s very clear how `Heater`, `Motor` and `VacuumCleaner` can be plugged into this system without changing any code in `Switch` at all. That is extensibility, reliability and reusability being delivered, right there.

The inversion (I in DIP) comes from the fact that `Lamp` (a low-level module) now depends on `ISwitchableDevice`, whereas originally it was `Switch` (a high-level module) which depended on `Lamp` (a low-level module).

Earlier, I promised I’d revisit the concept of “ownership” and "driving changes" into something.  We can see that a `Lamp` with a fancy dimming feature shouldn't change the `SwitchableDevice` interface.  If it did, then nothing would have changed from a code stability point of view.  Changes to `Lamp`'s interface would drive change in `ISwitchableDevice`'s interface, all its other implementations, and all users of the `ISwitchableDevice` API.  But it (should be) fairly clear that `Lamp` shouldn't be impacting `ISwitchableDevice`'s interface like that--if only by the interface's name.  Note that is is not so obvious (or even counterintuitive) if `Lamp` is its own interface (or implements a trivial abstraction like, say `ILamp`).

Instead, consider packaging the abstracted interface with the high-level modules that depend on it. This makes it clearer that the interface does not “belong” to the implementations. Any changes to the interface should be driven by the needs of the interface's clients (high-level module(s)).

If multiple high-level modules are making use of the same interface, then the interface should be packaged on its own (no single “owner”). Either strategy preserves the concept that the implementation shouldn’t own its interface.
