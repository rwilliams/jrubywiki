Google Summer of Code 2013
=========================

This page hosts the ideas for Google Summer of Code 2013! Add your ideas here, improve others, and if you're a student, perhaps something on this list will interest you!

Communication
=============

JRuby's GSoC communications take place (at least initially) on the [jruby-gsoc](https://groups.google.com/forum/#!forum/jruby-gsoc) Google Group. Once projects get going, most mentors and students will take their discussions offline, but while we're preparing for the summer most conversations happen on that list.

We also strongly encourage interested students and mentors to start communicating with the JRuby team as early as possible, either on the JRuby dev mailing list or in our Freenode #jruby IRC channel.

Ideas
=====

Here's some classic ideas to get you started:

## Native libraries that need a Java port 

* or wrap a Java lib?

* @headius's [wrapper around spymemcached](https://github.com/headius/jruby-spymemcached) needs a nice compatible Ruby API.

* The Ragel-generated JSON gem for JRuby is currently slower than C versions because Ragel does not generate gotos (since Java has no gotos). Investigate ways to improve perf, possibly by adding JVM bytecode support (JVM bytecode has goto) to Ragel.

* [jruby-openssl](https://github.com/jruby/jruby-ossl) needs better compatibility with MRI's implementations

* [eventmachine's](https://github.com/eventmachine/eventmachine) TLS/SSL support is missing, currently just stubbed out

## JRuby on Android: Ruboto

Ruboto is working, and has a solid IRB application and tools for generating apps. But there's more we can do, like shrinking the app, improving performance, and building better tooling.

* *Improve startup time*.  Anything done here will make Ruboto more usable.
* *Reduce runtime size*. Find ways to reduce the amount of .class data we ship to devices.
* *Straight-line performance*. Android devices are very resource-limited, and we would like to improve the performance of JRuby on Android to address this fact.

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