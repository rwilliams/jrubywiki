Concurrency in JRuby
====================

JRuby was the first Ruby implementation to offer true concurrency, due to the fact that in JRuby a Ruby thread is just a Java thread, and most JVMs map Java threads directly to native threads. As a result, JRuby has set the standard for what it means to do concurrent programming in Ruby.

This page will contain information on JRuby's memory model, strategies for helping you write safe concurrent code in JRuby, and patterns to watch out for.

Concurrency Basics
------------------

JRuby maps Ruby threads to Java threads, which are usually mapped directly to native threads. This means a simple Ruby ```Thread.new { }``` produces a real OS thread that runs in parallel with the parent thread.

JRuby provides the same concurrency primitives as standard Ruby in the 'thread' library (loaded by default in 1.9 mode). Mutex, ConditionVariable, Queue, and friends all work as they do in MRI, but they are often crucial to writing threadsafe code in JRuby.

In general, the safest path to concurrency in JRuby is the same as on any other platform:

1. Don't do it.
2. If you must do it, don't share data across threads.
3. If you must share data across threads, don't share mutable data.
4. If you must share mutable data across threads, synchronize access to that data.

Outside the Ruby specifics described below, JRuby operates within the confines of the
[Java Memory Model](http://www.cs.umd.edu/users/pugh/java/memoryModel/jsr-133-faq.html).

Thread Safety
-------------

Thread Safety refers to the ability to perform operations against a shared structure across multiple threads
and know there will be no resulting errors or data integrity issues.

The JRuby runtime itself is considered to be threadsafe. From Java, you can use a single runtime safely across threads, provided the code in those thread does not do thread-unsafe Ruby operations listed below. From Ruby, this means that the JRuby runtime itself will never be corrupted by concurrent operations.

More specifically, the following operations are thread-safe in JRuby:

* Opening or reopening a class. Note that there still may be races if two threads attempt to edit the same class in the same way at the same time.
* Defining or modifying methods. The same race conditions apply, but a given class's method table will never be corrupted by concurrent modifications.
* Defining or modifying constants and class variables.
* Defining or modifying global variables.
* Requiring libraries. Note that the code in those libraries may race, if for example they try to modify the same classes. But concurrent requires will never leave the JRuby runtime in an inconsistent state. Additionally, in 1.9 mode only one thread can be in the process of loading a given file.
* Autoloading libraries. A given autoload is only allowed to run in a single thread, and others that encounter the autoload before completion will block.
* Updating instance variables. The race in this case is more subtle; if new instance variables are being defined for the first time in multiple threads, it's possible for one thread to throw away the results of another. If an object is going to be used across multiple threads, we recommend you initialize its instance variables in the #initialize method.

JRuby does not, however, make thread-safety guarantees about several core classes, primarily because introducing thread-safety (through locking) would negatively impact all non-threaded use of these structures.

For all these cases, if an object ends up in an inconsistent state, either JRuby or the JVM will raise an exception. We endeavor to make those JRuby exceptions as often as possible.

At least these classes are not considered thread-safe, and if you intend to mutate them concurrently with other operations you will want to introduce locking (e.g. with Mutex): String, Array, Hash, and any data structures derived from them.

Volatility
----------

Volatility refers to the visibility of changes across threads on multi-core systems that may have thread or core-specific views of system memory.

Modern CPUs use multi-level caches to avoid reads and writes to main memory, which are considerably slower than reads and writes to on-die memory. In multi-core systems, these caches can often become inconsistent across cores. Volatility is a way to specify to the runtime or the CPU that a given memory operation should be synchronized across caches.

JRuby uses the JVM's implementation of volatility for some features, and leaves others nonvolatile. Specifically, the following operations are volatile:

* Modifying a class's method table.
* Modifying a class's constant table.
* Modifying a class's class variables (not to be confused with class instance variables).
* Modifying a global variable.
* Growing an object's instance variable table (though the entries are not volatile).
* Using thread-synchronization primitives like Mutex.

The following operations are not volatile, and there is potential for threads to see inconsistent views of memory.

* Updating an instance variable.
* Modifying any of the non-threadsafe core data structures.
* Modifying local variables across threads, as in a closure used in a parallel-execution setting.

If you wish to ensure you are making volatile updates to a reference, we recommend using the ```atomic``` gem to hold that reference. It is described in more detail below.

Atomicity
---------

Atomicity refers to the ability to perform a write to memory based on some view of that memory and to know the write happens before the view is invalid. There is a strong parallel here with volatility. An example would be updating a value if and only if it is still null. See the important not about the *lack* of atomicity in Ruby's ||= operation below.

Very few operations in JRuby have any guaranteed atomicity. Usually, this is expected; most operations are done as simple reads or unconditional writes, like constant initialization, method definition, and so on. A few operations, however, are done atomically:

* Growing an instance variable table. This generally only comes into play early in execution, or if new instance variables are defined for the first time under concurrent execution.
* Updates of LOADED_FEATURES ($") in response to concurrent requires. If a feature appears in LOADED_FEATURES, you know it has successfully completed loading in exactly one thread.
* The state of an autoloaded constant (1.9 mode only). Autoloads will only run and complete in exactly one thread.

A number of common Ruby features, however, are *not* guaranteed atomic, even though they may imply such.

* Conditional updates of the form ```||=``` or ```&&=```. These are not actually atomic in any version of Ruby, and expand logically to a read followed by a write. Additionally, there's no guarantee the right-hand side (the value expression) will only execute once.
* Updates with modification, as in ```+=```, ```-=``` and friends. These expand to a read, method call, and write.

If you need atomic operations, we recommend using the ```atomic``` gem. The ```atomic``` gem provides a number of operations for doing atomic updates:

* compare_and_set - only set the value if it is currently set to some and expected value.
* get_and_set - get the current value and set a new one in a single operation.

In addition, there are simple get and set operations for achieving simple volatility, as well as block-receiving forms of compare_and_set that will use the block's value -- retrying it as necessary -- until the update is successful.

Additional Support
------------------

Because JRuby is on the JVM, you also have access to all that the JVM offers in terms of concurrency. The ```java.util.concurrent``` package contains a number of threadsafe collections, queues, sets, and other data structures. The ```java.util.concurrent.atomic``` package provides atomic containers for reference and primitive types. And there are many actor libraries, transactional memory libraries, and other concurrency utilities available to you.

JRuby also provides a non-standard way to convert any object or class into a fully-synchronized data structure: the ```JRuby::Synchronized``` module. You can simply require ```require 'jruby/synchronized'``` and then ```include JRuby::Synchronized``` into any class or ```obj.extend(JRuby::Synchronized)``` any object. The result is that all methods on that class or object will be wrapped in simple synchronization; that is, a reentrant lock against the object itself.