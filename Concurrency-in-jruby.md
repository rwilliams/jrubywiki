Concurrency in JRuby
====================

JRuby was the first Ruby implementation to offer thread-level parallelism, due to the fact that in JRuby a Ruby thread is just a Java thread, and most JVMs map Java threads directly to native threads. As a result, JRuby has set the standard for what it means to do concurrent programming in Ruby.

This page will contain information on JRuby's memory model, strategies for helping you write safe concurrent code in JRuby, and patterns to watch out for.

Table of Contents
-----------------

* [Concurrency Basics](#concurrency_basics)
* [Thread Safety](#thread_safety)
* [Core Classes and the Standard Library](#core_classes_standard_library)
* [Volatility](#volatility)
* [Atomicity](#atomicity)
* [Additional Support](#additional_support)

<a name="concurrency_basics">
Concurrency Basics
------------------

JRuby maps Ruby threads to Java threads, which are usually mapped directly to native threads. This means a simple Ruby `Thread.new { }` produces a real OS thread that runs concurrently with the parent thread.

JRuby provides the same concurrency primitives as standard Ruby in the 'thread' library (loaded by default in 1.9 mode). Mutex, ConditionVariable, Queue, and friends all work as they do in MRI, but they are often crucial to writing threadsafe code in JRuby.

In general, the safest path to writing concurrent code in JRuby is the same as on any other platform:

1. Don't do it, if you can avoid it.
2. If you must do it, don't share data across threads.
3. If you must share data across threads, don't share mutable data.
4. If you must share mutable data across threads, synchronize access to that data.

Outside the Ruby specifics described below, JRuby operates within the confines of the
[Java Memory Model](http://www.cs.umd.edu/users/pugh/java/memoryModel/jsr-133-faq.html), and this document uses the same terminology.

<a name="thread_safety">
Thread Safety
-------------

Thread Safety refers to the ability to perform operations against a shared structure across multiple threads and know there will be no resulting errors or data integrity issues.

### Language Features and the JRuby Runtime

The JRuby runtime itself is considered to be threadsafe. From Java, you can use a single runtime safely across threads, provided the code in those threads does not do thread-unsafe Ruby operations listed below. From Ruby, this means that the JRuby runtime itself will never be corrupted by concurrent operations. Thread-safety does *not* mean your code will always run correctly; you will still often need to ensure threads don't step on each others' modifications.

More specifically, the following operations are thread-safe in JRuby:

* [Opening (or re-opening) a class](https://github.com/jruby/jruby/wiki/Concurrency in JRuby%3A Opening a class)
* [Call site invalidations](MethodTableConcurrency)
* Defining or modifying methods.
* Defining or modifying constants and class variables.
* Defining or modifying global variables.
* Requiring libraries.
* Autoloading libraries (JRuby 1.7+). A given autoload is only allowed to run in a single thread, and others that encounter the autoload before completion will block.
* Updating instance variables.

Note that thread safety in this context only means that concurrent updates to the same runtime data structure managed by the JRuby runtime are serialized in some order -- JRuby cannot guarantee a specific order in which those updates will be performed across threads (Ex: [Opening a class](https://github.com/jruby/jruby/wiki/Concurrency in JRuby%3A Opening a class)).  User code is responsible for enforcing specific ordering requirements where such ordering is important for the program's correct execution.  For example, two libraries required at the same time that try to define the same methods or constants -- will still update the relevant class, but the specific order in which those updates are performed will determine the final state of that class.  JRuby does not (and cannot) make any specific ordering guarantees for such concurrent modifications (or for any non-atomic concurrent modifications throughout your code).

You should take care in the following situations: concurrent requires, or lazy requires that may happen at runtime in a parallel thread. If objects are in flight while classes are being modified, or constants/globals/class variables are set before their referrents are completely initialized, other threads could have problems.


<a name="core_classes_standard_library">
### Core Classes and Standard Library

JRuby does not, however, make thread-safety guarantees about several core classes, primarily because introducing thread-safety (through locking) would negatively impact all non-threaded use of these structures.

For all these cases, if an object ends up in an inconsistent state, either JRuby or the JVM will raise an exception. We endeavor to make those JRuby exceptions as often as possible.

At least these classes are not considered thread-safe, and if you intend to mutate them concurrently with other operations you will want to introduce locking (e.g. with Mutex): String, Array, Hash, and any data structures derived from them.

JRuby reuses the same standard library (the .rb files we ship but which you must require, like ```date``` or ```net/http```), and does not generally alter those libraries' thread-safety guarantees. Unfortunately, the thread-safety of many of these libraries is unknown at this time, though we do not have any outstanding bug reports of thread-unsafe standard libraries at this time.

Notable exceptions are libraries which we have rewritten or replaced with pure-Java versions: ```thread```, ```weakref```, ```generator```, ```timeout```, and ```fcntl``` have been partially or completely rewritten, and are expected to be thread-safe.

<a name="volatility">
Volatility
----------

Volatility refers to the visibility of changes across threads on multi-core systems that may have thread or core-specific views of system memory.

Modern CPUs use multi-level caching to reduce main memory accesses, since reads and writes to main memory are orders of magnitude slower than reads and writes to on-chip caches. In multi-core systems, these caches may become inconsistent across cores, or runtimes and compilers may take liberties with the ordering of those reads and writes. Volatility is a way to specify to the runtime or the CPU that a given memory operation should be synchronized across caches and that those reads and writes must happen in an explicit order.

JRuby uses the JVM's implementation of volatility for some features, and leaves others nonvolatile. Specifically, the following operations are volatile:

* Modifying a class's method table.
* Modifying a class's constant table.
* Modifying a class's class variables (not to be confused with class instance variables).
* Modifying a global variable.
* Growing an object's instance variable table (though the entries are not volatile).
* Using thread-synchronization primitives like Mutex.

The following operations are not volatile, and there is potential for threads to see inconsistent views of memory.

* Modifying any of the non-threadsafe core data structures.
* Modifying local variables across threads, as in a closure-captured local variable used in a parallel-execution setting.

Instance variables in JRuby are currently volatile, but we have been reluctant to make that a hard guarantee. Volatility is achieved in one of three ways:

1. On JVMs that do not have or do not allow access to sun.misc.Unsafe, we use full JVM-level synchronization. This synchronization is only around the actual access of the instance variable, and generally should not be a major choke point.
2. On Java 6 and 7, we use Unsafe.putOrderedObject and putObjectVolatile to update the variable table reference and the table entries, respectively.
3. On Java 8, we use Unsafe.putOrderedObject to update the variable table reference and Unsafe.fullFence to insert an explicit memory barrier after table writes.

Both (2) and (3) above also use a volatile numeric stamp to ensure variable table reference changes are done atomically.

Note that volatility does not guarantee atomicity; two threads updating a variable and a third observing them may see the writes in any order. If you need atomicity (or if you need volatility for a reference that is not otherwise guaranteed to be volatile), we recommend using the ```concurrent-ruby``` gem, which provides explicit reference types that provide both volatility and atomic operations. Atomicity is described in more detail below.

<a name="atomicity">
Atomicity
---------

Atomicity refers to the ability to perform a write to memory based on some view of that memory and to know the write happens before the view is invalid. There is a strong parallel here with volatility. An example would be updating a value if and only if it is still null. See the important note about the *lack* of atomicity in Ruby's `||=` operation below.

Very few operations in JRuby have any guaranteed atomicity. Usually, this is expected; most operations are done as simple reads or unconditional writes, like constant initialization, method definition, and so on. A few operations, however, are done atomically:

* Growing an instance variable table. This generally only comes into play early in execution, or if new instance variables are defined for the first time under concurrent execution. See above for a description of how we make this guarantee.
* Updates of `LOADED_FEATURES` ($") in response to concurrent requires. If a feature appears in `LOADED_FEATURES`, you know it has successfully completed loading in exactly one thread.
* (from JRuby 1.7.x) The state of an autoloaded constant. Autoloads will only run and complete in exactly one thread.

A number of common Ruby features, however, are *not* guaranteed atomic, even though they may imply such.

* Conditional updates of the form `||=` or `&&=`. These are not actually atomic in any version of Ruby, and expand logically to a read followed by a test and potentially a write. Additionally, there's no guarantee the right-hand side (the value expression) will only execute once.
* Updates with modification, as in `+=`, `-=` and friends. These expand to a read, method call, and write. Under concurrency, the write may wipe out another thread's update.

If you need atomic operations, we recommend using the `concurrent-ruby` gem. The [concurrent-ruby](https://github.com/ruby-concurrency/concurrent-ruby) gem provides several atomic classes supporting a number of operations for doing atomic updates:

* compare_and_set - only set the value if it is currently set to some and expected value.
* get_and_set - get the current value and set a new one in a single operation.

In addition, there are simple get and set operations for achieving simple volatility, as well as block-receiving forms of compare_and_set that will use the block's value -- retrying it as necessary -- until the update is successful.

<a name="additional_support">
Additional Support
------------------

Because JRuby is on the JVM, you also have access to all that the JVM offers in terms of concurrency. The `java.util.concurrent` package contains a number of threadsafe collections, queues, sets, and other data structures. The `java.util.concurrent.atomic` package provides atomic containers for reference and primitive types. And there are many actor libraries, transactional memory libraries, and other concurrency utilities available to you.

JRuby also provides a non-standard way to convert any object or class into a fully-synchronized data structure: the `JRuby::Synchronized` module. You can simply require `require 'jruby/synchronized'` and then `include JRuby::Synchronized` into any class or `obj.extend(JRuby::Synchronized)` any object. The result is that all methods on that class or object will be wrapped in simple synchronization; that is, a reentrant lock against the object itself.

`JRuby::Synchronized` is used to within the [`concurrent-ruby`](https://github.com/ruby-concurrency/concurrent-ruby) gem, which provides `Synchronization::Object`, `Concurrent::Array`, `Concurrent::Hash`, and many other concurrency abstractions for use in JRuby (and MRI).