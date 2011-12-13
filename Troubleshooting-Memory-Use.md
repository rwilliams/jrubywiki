Because JRuby runs on the JVM, the memory profile of a typical Ruby application will differ from what you might normally see under the standard C implementation of Ruby. This page discusses a few reasons for this difference and what you can do about it (when there's something that needs to be done, at least).

Some tips have been collected on [[Improving Memory Consumption|Improving-Memory-Consumption]].

Generational GC likes memory
============================

The standard implementation of Ruby has what's called a conservative garbage collector. A single large memory space is used for all objects, and GC involves walking that memory space, looking for pointers that are no longer in use, and reusing those slots in the memory heap. While there are down sides to this approach (linearly slower GC times for larger heaps, potential memory fragmentation, etc), one benefit is that your memory footprint is usually only a bit larger than maximum size of all objects alive at once.

Newer Ruby runtimes usually have some form of generational garbage collection. In a generational collector, the memory space used by the runtime is divided up by how long an object has been around. The youngest objects are allocated in the young generation area of the heap, and as they age they're moved to older generations. Because the young area of the heap stays relatively small, GC times for even large and busy applications compares well to small applications. However, this also means that even an optimistic generational garbage collector will use more memory than a conservative collector, since the ideal case is to have lots of room to promote or "tenure" objects into different areas of memory.

Comparisons of raw process size between JRuby and other conservative-GC'ed implementations can therefore be problematic.

JVM uses whatever you give it
=============================

Most JVMs have a notion of a maximum heap size, wherein you can specify a size the VM shall never exceed, and if an applications tries to force the VM past that limit an error will be thrown. However within that maximum size, the JVM is free to use as much space as it sees fit; this often means that even large limits will eventually be allocated by the process in order to give the garbage collector and memory management logic as much room as possible to work with...so-called "breathing room".

JRuby sets the default maximum heap size to around 500MB. This was a number found through experimentation with various libraries and applications, and has been set to give plenty of room for libraries/tools that consume larger amounts of memory (RubyGems on boot, Rails at runtime). It's obviously not a one-size-fits-all number, though, since most Ruby applications can run with a lot less than 500MB.

Unfortunately, the JVM sees the 500MB limit and often grows the heap to nearly that size if it seems like a larger heap will make GC more efficient. This often leads to the mistaken perception that JRuby uses a lot more memory, when really all it is doing is taking advantage of more memory to run better.

You can force the underlying JVM back down to a smaller heap size by passing -J-Xmx###m to JRuby, where #### represents some size in mb (indicated by the final 'm'), which often will trim the size of your running process.

Rails apps are often not threadsafe and usually aren't running in threadsafe mode
=================================================================================

The typical way to deploy a Rails application is to deploy N instances for the N concurrent requests you want to be able to handle at the same time. On MRI, this is accomplished by simply launching N processes (perhaps with a few extra for wiggle room) that all contain the same application code and are fronted by some sort of balancing proxy. On JRuby, a single process is used, but we start up N JRuby instances inside a single JVM, simulating the multi-process model.

Rails 2.2 brought with it something called "threadsafe" mode. If you turned on "threadsafe", certain lazily-initialized data structures would be eagerly initialized, and various other concurrency-unfriendly parts of Rails were wrapped with appropriate synchronization. The result is that many Rails applications, properly written, can run as a single instance on JRuby and handle as much (or more) concurrent load as N instances.

We generally recommend getting your application "threadsafe", since in a much smaller memory footprint you can handle a tremendous number of concurrent requests. However, most JRuby deployment options still default to non-threadsafe "N instance" models. This often explains why a Rails application atop JRuby consumes way more memory out of the box than the same application atop MRI; you're comparing a whole cluster (JRuby's N instances in one VM) with one member of a cluster (MRI's model of process-per-instance).

The long term way to reduce memory here is to make your application threadsafe and run it in that mode. Barring that, you can usually (using server-specific flags) specify how many JRuby instances (min and max) should be launched. For example, if you never need to handle more than one concurrent request, you may only ever need to launch one JRuby instance.

JRuby's more OO
===============

Because JRuby is written mostly in Java (rather than C), the entirety of the JRuby runtime is implemented in a very typical object-oriented way. This makes it a more approachable codebase, but all that composition can have a memory cost when compared to flat, dumb C structs that are only as large as their contents. Better abstractions often cost memory (or performance).

We try as much as possible to avoid excessive object size, but some of this difference is unavoidable.

JRuby's not perfect!
====================

There are certainly areas where JRuby might use more memory simply because parts of JRuby have not yet been optimized for memory use or because there's a bug causing us to use too much memory (or worse, to leak memory in hard or soft ways). If you run into such a situation and none of the above remedies seems to help, you should report it as a bug at [[http://bugs.jruby.org]].