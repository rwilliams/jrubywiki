Google Summer of Code 2013
=========================

This page hosts the ideas for Google Summer of Code 2013! Add your ideas here, improve others, and if you're a student, perhaps something on this list will interest you!

Practical Details
=================

The [JRuby organization's page](https://google-melange.appspot.com/gsoc/org/google/gsoc2013/jruby) in the Google Summer of Code app will let you apply as a mentor or student. If you can't be a mentor or student, we hope you will pass this information on to others.

### Goals

Of course we'd love to have a ton of great JRuby work come out of GSoC, but we'd also like that work to benefit the larger Ruby community where possible. Many of the ideas listed below apply equally to JRuby and other implementations.

We also hope that students will continue to be JRuby contributors long after GSoC 2013 is complete. We're looking to help students grow their resumes and help the JRuby project grow its community.

Our primary focus is on making sure students get a good summer's work and education out of the project.

### Dates

The full, official calendar is here: https://google-melange.appspot.com/gsoc/events/google/gsoc2013

* April 9 - 21: Mentors can apply and students should start talking with mentors and JRuby team members about ideas.

* April 22 - May 3: Students can apply to work on a JRuby GSoC project.

* May 4 - 6: Mentors review and accept student proposals.

* May 8: GSoC provides JRuby with a count of slots for student projects.

* May 9-22: Slot juggling; available slots are assigned to student projects.

* May 24: Final decisions on slots and students.

* May 28 - June 16: Students get familiar with their mentors and start planning out the summer. *NOTE:* We expect students to start talking with mentors and JRuby team members *long* before this time.

* June 17: Students begin work on their projects. Starting before this date is encouraged :-)

* July 29 - August 2: Midterm reviews; mentors grade students on progress thus far.

* September 16: Soft "pencils down" date. Students should be finished or nearly-finished with their projects.

* September 23: Hard "pencils down" date. Students must be done with the official GSoC portion of their project. We *strongly* encourage students to continue working with the JRuby community after GSoC, but this ends the graded portion of the project.

* Sepember 23 - 27: Mentors submit final reviews of student projects.

Communication
=============

JRuby's GSoC communications take place (at least initially) on the [jruby-gsoc](https://groups.google.com/forum/#!forum/jruby-gsoc) Google Group. Once projects get going, most mentors and students will take their discussions offline, but while we're preparing for the summer most conversations happen on that list.

We also strongly encourage interested students and mentors to start communicating with the JRuby team as early as possible, either on the JRuby dev mailing list or in our Freenode #jruby IRC channel.

Ideas
=====

Here's some classic ideas to get you started:

## Rails Performance

Over the years, JRuby has gotten better and better at running Rails, but there remains work to do. This project would involve gathering existing benchmarks and "standard" applications (Redmine, etc) and using them to find remaining perf issues in JRuby. Some possible areas that could use improvement:

* ActiveRecord-JDBC, the library that JRuby uses to wrap JDBC with ActiveRecord's ORM API. We have done various performance investigations over the years, but a concentrated effort to make it consistently faster across databases has never really been attempted.

* jruby-rack and other servers. A JRuby on Rails app can only serve requests quickly if the server frontend is fast. There's always more that we can do to improve the speed of these requests, in jruby-rack (used for serving JRuby in existing app servers like Tomcat) or in purpose-built servers like Torquebox (JRuby for JBoss), Puma, Passenger, and others.

* Data format libraries like json and yaml. More and more users of Rails are building JSON-based RESTful APIs atop it, and again the performance of one library (json) can be the limiting factor.

* Templating libraries like Haml. In order to render results quickly to the browser, these libraries need to be fast. Haml in particular employs a number of Ruby code patterns that can severely impact performance. We need to investigate the performance of all these libraries and ensure that we run them as fast as possible.

* Other libraries commonly used in Rails apps, such as for accessing caching servers like memcached, nosql databases like mongo, and queues like zeromq.

## Native libraries that need a Java port 

* or wrap a Java lib?

* @headius's [wrapper around spymemcached](https://github.com/headius/jruby-spymemcached) needs a nice compatible Ruby API.

* The Ragel-generated JSON gem for JRuby is currently slower than C versions because Ragel does not generate gotos (since Java has no gotos). Investigate ways to improve perf, possibly by adding JVM bytecode support (JVM bytecode has goto) to Ragel.

* [jruby-openssl](https://github.com/jruby/jruby-ossl) needs better compatibility with MRI's implementations

* [eventmachine's](https://github.com/eventmachine/eventmachine) TLS/SSL support is missing, currently just stubbed out

## Ruboto: JRuby on Android

Ruboto is working, and has a solid IRB application and tools for generating apps. But there's more we can do, like shrinking the app, improving performance, and building better tooling.

* *Improve startup time*.  Anything done here will make Ruboto more usable.
* *Reduce runtime size*. Find ways to reduce the amount of .class data we ship to devices.
* *Straight-line performance*. Android devices are very resource-limited, and we would like to improve the performance of JRuby on Android to address this fact.
* *JIT compilation*.  Increase execution speed and reduce stack usage.
* *AOT compilation*.  Faster startup and increased execution speed.

## JRuby for Embedded

There's a few good JVMs that work on embedded devices, which means there's an opportunity for JRuby to expand into embedded applications.

## Maven support for Rubygems, Bundler, etc

JRuby has great java integration but it's a pain having to manage java dependencies manually. Making Rubygems and Bundler aware of Maven on JRuby would be awesome!

## JRuby and Vert.x improvements

Vert.x provides an evented environment very similar to node.js, but without many of its flaws. JRuby is supported, but performance is not where we'd like it. There's a good summer's worth of work on better integrating JRuby and Vert.x so they can act as a high-performance evented Ruby environment.

## JRuby IR-based projects

JRuby currently has an intermediate representation (IR) that attempts to capture high-level Ruby semantics via instructions and operands.  This IR will be the basis of an updated JRuby VM.  While some optimizations are already in place including dead code elimination, method and block inlining (incomplete), there are lots of opportunities for improving on these and implementing additional optimizations.  A student interested in interpreters, compilers, virtual machines would work with the JRuby team to expand on the capabilities of this VM -- projects could include work on the interpreter, new performance optimizations, implementing new backends (ex: Dalvik).

A couple sample projects:

* Optimize IR for interpretation: The IR is optimized for performance optimizations and for the JIT which want explicit state in IR, complex instructions broken down into simpler primitive instructions.  But, interpreter wants fewer instructions since each extra instruction adds to interpretation overhead. Implement IR transformations that optimize IR right before interpretation (ex: collapse instruction chains within a basic block into expression trees, collapse multiple simple instructions into single complex instructions, etc.).  Some of this functionality is in place in bits and pieces in very preliminary form, but it has not been integrated, tested, thought through.

* Target a different backend (ex: Dalvik): Investigate what it would take to compile IR to the Dalvik platform, and implement it within the constraints of a 3-month project.

* IR compiler optimizations: The JRuby IR provides a great opportunity to optimize the Ruby language, both by statically analyzing code and by gathering runtime profiles. The next major version of JRuby is intended to run entirely atop the IR compiler, and so we are looking for compiler and optimization folks to help us layer incremental improvements on top of the base IR logic we have today.

## Shoes on JRuby

Shoes is a cross-platform toolkit for writing graphical apps easily and artfully using Ruby.
Unlike most other GUI toolkits, Shoes is designed to be easy and straightforward without losing power. Really, it’s easy!
Shoes needs you! The shoes community gathered to write [shoes4](https://github.com/shoes/shoes4) together. Shoes4 is a complete rewrite of the Shoes DSL allowing exchangeable GUI-backends. The first and default backend is using JRuby and SWT.
There are many interesting projects to tackle within the area of shoes4. You could work on video support, general support for more Shoes constructs or packaging stand alone applications.

## Celluloid "Turbo Mode" for JRuby

[Celluloid](http://celluloid.io) is an actor-based concurrent object framework (somewhat similar to Akka) written in pure Ruby. This means it presently uses Ruby Mutexes and ConditionVariables for synchronization. However, the JVM has many, many other options which could provide better performance.

The goal of this project would be to implement a duck type of the `Celluloid::Mailbox` class that leverages native JVM facilities to improve performance. Some examples to consider might be:

* ***[ArrayBlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ArrayBlockingQueue.html)***: These are fast, fixed-sized data structures built atop arrays. Their bounded size might require some semantic changes to Celluloid (see [this ticket for discussion on bounded mailboxes](https://github.com/celluloid/celluloid/pull/153)) but are probably the simplest way to improve performance on Celluloid.
* ***[LinkedTransferQueue](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedTransferQueue.html)***: Introduced in Java 7, LinkedTransferQueue could provide Celluloid's existing unbounded semantics with better performance than Java's previous linked queues. LinkedTransferQueues are a bit complicated and support lots of different modes of operation, so mapping them specifically to Celluloid's semantics might be a bit difficult.
* ***[LMAX Disruptor](http://lmax-exchange.github.io/disruptor/)***: Disruptor is a library which supports a number of different patterns for multithreaded execution. The main way LMAX could benefit Celluloid would be providing a way to preallocate and recycle inter-actor messages, storing them in a RingBuffer and providing cache-friendly operation while reducing the allocation rate and thus the demands on the GC. It's unclear if Disruptor's concurrency model could map to Celluloid's well, but it could be used in conjunction with the above data structures specifically for the purposes of leveraging preallocation.

## Java + Native subsystems

* JRuby native IO and Process APIs

JRuby currently uses Java's standard NIO and ProcessBuilder APIs for doing all IO and process management. We've managed to hack this well enough over the years, but there are many features we can't easily support using only Java's APIs.

The [Java Native Runtime](https://github.com/jnr) provides an [FFI layer](https://github.com/jnr/jnr-ffi) to Java that JRuby uses for [many basic POSIX functions](https://github.com/jnr/jnr-posix). One of its subprojects, [jnr-enxio](https://github.com/jnr/jnr-enxio), provides NIO-compatible wrappers around standard native IO operations, allowing things like selectable stdio (Java's stdio is not selectable), UNIX sockets ([jnr-unixsocket](https://github.com/jnr/jnr-posix)), and the potential for us to spawn subprocesses using functions like posix_spawn (which can carry parent descriptors through to children).

JRuby would benefit from work on a process-management and IO subsystem based on JNR, for cases where the Java APIs simply are not suitable.

* Additional platform support

JRuby supports calling native libraries across many platforms, but there are platforms we don't support. For example, we have never worked on or tested JRuby + JNI + FFI on Android, which could make all of the native APIs of Android available to JRuby-on-Android users. We're looking for folks interested in managed/unmanaged integration, dynamic language binding, and late-bound native invocation optimization to help us improve this situation.

## krypt

The Java parts of [krypt](https://github.com/emboss/krypt) ensure that JRuby no longer has to emulate the OpenSSL library. It uses Java's own JCE instead to implement a library-agnostic interface that provides full access to Ruby cryptography. If you are interested in cryptography in general, there is a wide variety of topics for you to work on - ranging from Authenticated Encryption modes, providing alternative implementations to [the JCE provider](https://github.com/emboss/krypt-provider-jce) on to XML or PDF signatures using Nokogiri for the former and Java open-source PDF libraries for the latter. If you are specializing in a particular topic and would like to apply it in reality, we'd enjoy to give you a playground to work on.

## Ruby 2.0 compatibility

[Ruby 2.0](http://www.ruby-lang.org/en/news/2013/02/24/ruby-2-0-0-p0-is-released/) is here.
That means JRuby needs to ensure that our libraries (both core and standard) are as compatible to it as possible.
The work will involve surveying MRI's libraries, writing missing [RubySpec](http://rubyspec.org/) specs, and implementing them.

## JRuby build cleanups / Mavenization

Improving the build/dist process overall.

Even if we already have maven support for the build, we still store some binaries and host a parallel ant build.

The idea is to continue improving that process, by moving the dependencies to maven central and in the ant build case, making it download the deps on the first run.

## Asciidoctor JRuby and Java integration

Asciidoctor is a Ruby implementation of AsciiDoc. It can be used as a full replacement for the Python implementation and also has support for running on JRuby. You can find the projects on the [Github Asciidoctor Organization](https://github.com/asciidoctor). Specifically we'd be looking at the asciidoctor-java-integration, asciidoctor-maven-plugin and the asciidoctor-gradle-plugin for consideration for GSoC 2013 projects.

Ideas include, improvements, new features, better testing, live preview and any others people can dream up.