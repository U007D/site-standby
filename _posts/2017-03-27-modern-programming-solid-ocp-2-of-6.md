---
layout: post
title:  "Modern Programming: S[O]LID (2 of 6)"
date:   2017-03-27 15:14:00 -0700
categories: modern-programming solid ocp engineering-excellence
---
# Introduction
This is part two of a series of articles on using a collection of principles known as SOLID to achieve higher-quality code, as measured primarily through its cost (or ease) of maintenance.

Part 1 of this series can be found [here](https://bradleygibson.github.io/modern-programming/solid/srp/engineering-excellence/2017/03/27/modern-programming-solid-srp-1-of-6.html).
# O - Open/Closed Principle (OCP)
For our purposes, this principle states that classes should be open to (allow) extension, but closed to (disallow) modification.  There is some confusion in the industry around this principle, because at the time this principle was first published, polymorphism was not yet widely in use; at the time, extending a software entity (whether class, function or module) typically meant editing and recompiling it.

It turned out that the principle was a good idea.  Fast-forwarding several years with the rise of more sophisticated object-oriented languages which did use polymorphism, it became clear that using OCP, one could inherit from a base class, extending its functionality **without** having to modify the base class, which directly correlates to fewer new bugs, and *zero* new bugs not directly related to the newly extended subclass.  

So the modern Polymorphic Open/Closed Principle is the flavor that I subscribe to (hereafter referred to simply as the Open/Closed Principle), suggests that the base class be made abstract, that is, it is either an interface or an abstract class, and cannot, itself, be instantiated.  This allows the interface (or the abstraction if you are closer to academia) to be used throughout the solution, and the implementation, or an extension of it, or a replacement for it to be used in a SOLID implementation without risk of undesired or surprise side-effects.

Here’s an example, with a simple `Logger` class which lets the caller log to different destinations (C#):

    public class Logger
    {
        public Logger Log(string message, LogType logType)
        {
            switch(logType)
            {
                case LogType.Console:
                    //Write message to console
                    break;
    
                case LogType.LocalDisk:
                    //Write message to local disk
                    break;
    
                default:
                    //throw (logType not recognized)
            }
            return this;
        }
    }

Now let’s add a new logging destination (`LogType`).  Note we must go update the Logger class, which means we risk altering or breaking existing behavior anywhere in the application which logs.  Further, anyone wanting to add a LogType must fully understand the existing implementation before adding new functionality or risk introducing bug which could manifest nearly anywhere in the application.

Contrast this with a refactored version of Logger subscribing to OCP:

    public enum LogType
    {
        Console,
        LocalDisk
    }
    
    public interface ILogger
    {
        ILogger Log(string message, LogType logType);
    }
    
    public class ConsoleLogger : ILogger
    {
        public ILogger Log(string message, LogType logType)
        {
            //Write message to console
            return this;
        }
    }
    
    public class LocalDiskLogger : ILogger
    {
        public ILogger Log(string message, LogType logType)
        {
            //Write message to local disk
            return this;
        }
    }

You can see that ILogger represents the interface that OCP calls for.  Adding a new type of Logger is now simply a matter of implementing the ILogger interface.  There is no chance that the new derivation of ILogger will break an existing implementation; the implementer is also no longer required to wade through and understand the existing implementations in advance of safely adding the new functionality.

In short, following this principle allows one to make significant changes to program behavior, and one can be confident those changes will not break existing functionality.  This is how OCP directly addresses our goal of achieving highly maintainable and extensible software.
