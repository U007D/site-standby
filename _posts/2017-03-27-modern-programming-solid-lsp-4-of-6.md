---
layout: post
title:  "Modern Programming: SO[L]ID (4 of 6)"
date:   2017-03-27 16:10:00 -0700
categories: modern-programming solid lsp engineering-excellence
---
# L - Liskov Substitution Principle (LSP), part ii

As we discussed in [Liskov Substitution Principle (LSP) part i](https://bradleygibson.github.io/modern-programming/solid/lsp/engineering-excellence/2017/03/27/modern-programming-solid-lsp-3-of-6.html), this installment will look at an example of LSP in code (C++ this time).

Classic examples for illustrating LSP are the [Circle-Ellipse](https://en.wikipedia.org/wiki/Circle-ellipse_problem) or [Rectangle-Square](https://en.wikipedia.org/wiki/Liskov_substitution_principle#A_typical_violation) problem. But I find [Tom Dalling’s variation](http://www.tomdalling.com/blog/software-design/solid-class-design-the-liskov-substitution-principle/) to be a gentler, yet just-as-effective introduction to the concept. (I’ve reproduced and summarized Tom’s post here to eliminate the risk posed by [link rot](https://en.wikipedia.org/wiki/Link_rot).)

Consider a program designed to render the patterns birds make flying in the sky. Let's assume the author wisely employs SOLID’s OCP principle to ensure and close (fix) basic common capabilities in the abstract (ie. non-instantiatable) base class:

    class Bird
    {
    public:
        virtual void setLocation(double longitude, double latitude) = 0;
        virtual void setAltitude(double altitude) = 0;
        virtual void draw() = 0;
    };

Our author allows for extensibility (for example, different flight patterns for different species of birds) through inheritance. Version 1 of the app goes to market and is a huge hit. The author hires more developers and version 2 goes out and does even better. Then, some movie comes out with cute, cuddly penguins and version 3 must have penguins. No problem, just create a Penguin class which inherits from Bird. But the problem becomes apparent when defining the implementation for `Penguin::Altitude`:

    void Penguin::setAltitude(double altitude)
    {
        //altitude can't be set because penguins can't fly
        //this function does nothing
    }

At first, this might not even seem like a problem--the interface may do nothing, but it is implemented--aren’t we OK? The answer is no, because `setAltitude()`’s behavior has been altered. A caller changing the altitude would expect the altitude to reflect the change; tests should break, rendered patterns would look weird as Penguins fluttered on the ground, possibly even colliding as they tried to occupy the same location in space (since they now lack an altitude dimension).

Despite our author’s best efforts to follow OCP, he failed LSP because he made the basic assumption that all birds can fly. The whole point of using the abstract base class in the first place was to be able to drop in new subclass minimizing risk, since it would be plugged into existing, working code. Because of that assumption, existing code must be modified to accommodate unique Penguin behavior. Not good.

So what’s the solution?

    //Solution #1: The wrong way to solve the problem
    void ArrangeBirdInPattern(Bird* aBird)
    {
        //dynamic_cast<type>(expr) is C++’s runtime-checked cast
        //(if cast fails && type is a pointer, returns null)
        Pengiun* aPenguin = dynamic_cast<Pengiun*>(aBird);
                                         
        if(aPenguin)
        {
            ArrangeBirdOnGround(aPenguin);
        }
        else
        {
            ArrangeBirdInSky(aBird);
        }
    }
OK, figure out if the Bird is a Penguin, and if it is, arrange it using the special “ground-aware” positioning algorithm. At first glance, this may seem to be a good solution. But LSP says that an entity should be able to work without knowing the actual derived class of the object. We are violating that explicitly here, by special-casing for the Penguin derived class. As we think about maintenance, what happens when we also add an Ostrich and an Emu, which also do not fly, but because they don't huddle like penguins, need their own "ground-aware" positioning algorithms?

Let’s try something else. We could add a predicate to tell us whether the bird in question was flightless or not:

    //Solution 2: A slightly better way to do it?
    void ArrangeBirdInPattern(Bird* aBird)
    {
        if(aBird->isFlightless())
        {
            ArrangeBirdOnGround(aBird);
        }
        else
        {
            ArrangeBirdInSky(aBird);
        }
    }

This is a bit better because `ArrangeBirdInPattern()` does not special case based on derived class--but it’s really just applying a bandage over the problem. Really, `isFlightless()` is just telling us whether we have “that problem” this particular object or not.

And we haven't really solved the problem for the case when different flightless birds need different "ground-arrangement" behavior, unless we repeat the above pattern and add predicates for `HuddlesInCold()` and `IsFiercelyTerritorial()`, etc. and have every Bird consumer in the app handle all these special cases.  This is still a clunky patch, which is likely to lead us down the Tight Coupling/Low-Cohesion Path of Pain.

What happens when we try addressing the flawed assumption itself?

    //Solution 3: Proper API design
    class Bird
    {
    public:
        virtual void draw() = 0;
        virtual void setLocation(double longitude, double latitude) = 0;
    };
    
    class FlightfulBird : public Bird {
    public:
        virtual void setAltitude(double altitude) = 0;
    };

Now the Bird base class does not prescribe any flight functionality; this is supplied by the FlightfulBird subclass. This means that software entities that do not require flight capability can utilize all Birds, while those that do have a dependency on flight can utilize all FlightfulBirds.  Similar interfaces can be made for other birds which require special behavior.  Note that because we have implemented our classes as pure virtual classes (aka Interfaces) in accordance with LSP, we can even mix and match behavior with `Flightful` `HuddlesInCold` birds, for example, without risk of the [issues](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) which can arise from use of C++'s multiple inheritance.

By separating out the correct behaviors (which, of course, depend entirely on the specific problem being solved), a system can be designed to be robust in the face of change by keeping the scope of that change well-compartmentalized, which, from the maintenance perspective, means easier changes, lower-cost changes or often, both.
