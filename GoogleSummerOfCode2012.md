Google Summer of Code 2012
=========================

This page hosts the ideas for Google Summer of Code 2012! Add your ideas here, improve others, and if you're a student, perhaps something on this list will interest you!

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
* *Use the new byte code generation libraries* to allow JRuby JIT compilation and subclassing Java classes.
* *Add tooling for AOT compilation*

## JRuby for Embedded

There's a few good JVMs that work on embedded devices, which means there's an opportunity for JRuby to expand into embedded applications.

## Kilim integration

Kilim provides a fast coroutine implementation that could be used to provide Fibers or other lightweight tasks inside JRuby. Kilim can now weave bytecode at runtime so integration with a language like JRuby might now be practical.

See http://jira.codehaus.org/browse/JRUBY-6408

## JRuby GUI library

JRuby can make use of existing Java GUI toolkits like Swing via the existing Java integration. It can be better. Ideas include resurrecting [monkeybars](http://monkeybars.rubyforge.org/) and building a wrapper for [JavaFX](http://javafx.com/).

## Maven support for Rubygems and Bundler

JRuby has great java integration but it's a pain having to manage java dependencies manually. Making Rubygems and Bundler aware of Maven on JRuby would be awesome!

## EventMachine on Netty

Netty is *the* Java NIO library of choice. Eventmachine has a java reactor, but it is out of date, doesn't have feature parity and doesn't have any active maintainers. It would be great to (a) expose a nice Netty + Ruby API, and (b) provide an "EM compatibility layer" to help migrate existing EM projects onto JRuby. 

## Thin on JRuby

If we get better Eventmachine support for JRuby, having a running Thin WebServer based on Eventmachine would be great.
Then it would be easier to run Eventmachine jobs from within a e.g. Rails application. Everything would run within the reactor and you did not have to run the EM reactor in a new thread.

## JRuby IR-based projects

JRuby currently has an intermediate representation (IR) that attempts to capture high-level Ruby semantics via instructions and operands.  This IR will be the basis of an updated JRuby VM.  While some optimizations are already in place including dead code elimination, method and block inlining (incomplete), there are lots of opportunities for improving on these and implementing additional optimizations.  A student interested in interpreters, compilers, virtual machines would work with the JRuby team to expand on the capabilities of this VM -- projects could include work on the interpreter, new performance optimizations, implementing new backends (ex: Dalvik).

A couple sample projects:

* Write/read IR to/from disk: This feature will enable JRuby to avoid parsing Ruby source, build AST, and generate IR across runs.  This feature might be especially useful for faster bootup of short-running programs (ex: tests).  This project would have to ensure that the (human-readable) textual output captures all necessary information from IR so that when it is read back in, the IR can be reconstructed without loss of information.  Additionally, this should happen transparently without programmer intervention, i.e. it may have to implement some form of timestamping to ensure that the IR output is not out-of-date w.r.t the source code.  It should be robust -- that is, failure to write/read IR output should not crash the runtime.

* Optimize IR for interpretation: The IR is optimized for performance optimizations and for the JIT which want explicit state in IR, complex instructions broken down into simpler primitive instructions.  But, interpreter wants fewer instructions since each extra instruction adds to interpretation overhead. Implement IR transformations that optimize IR right before interpretation (ex: collapse instruction chains within a basic block into expression trees, collapse multiple simple instructions into single complex instructions, etc.).  Some of this functionality is in place in bits and pieces in very preliminary form, but it has not been integrated, tested, thought through.

* Target a different backend (ex: Dalvik): Investigate what it would take to compile IR to the Dalvik platform, and implement it within the constraints of a 3-month project.

## Shoes on JRuby
Shoes is a cross-platform toolkit for writing graphical apps easily and artfully using Ruby.
Unlike most other GUI toolkits, Shoes is designed to be easy and straightforward without losing power. Really, it’s easy!
Shoes needs you!  There are three research implementations of Shoes on JRuby.  Your work will go toward merging the libraries, choosing the best GUI lib for Shoes (SWT, Swing, Gtk?), and implementing a startup framework that makes Shoes on JRuby truly cross-platform.

## Java Native Runtime-based IO and process-mgmt subsystem

JRuby currently uses Java's standard NIO and ProcessBuilder APIs for doing all IO and process management. We've managed to hack this well enough over the years, but there are many features we can't easily support using only Java's APIs.

The [Java Native Runtime](https://github.com/jnr) provides an [FFI layer](https://github.com/jnr/jnr-ffi) to Java that JRuby uses for [many basic POSIX functions](https://github.com/jnr/jnr-posix). One of its subprojects, [jnr-enxio](https://github.com/jnr/jnr-enxio), provides NIO-compatible wrappers around standard native IO operations, allowing things like selectable stdio (Java's stdio is not selectable), UNIX sockets ([jnr-unixsocket](https://github.com/jnr/jnr-posix)), and the potential for us to spawn subprocesses using functions like posix_spawn (which can carry parent descriptors through to children).

JRuby would benefit from work on a process-management and IO subsystem based on JNR, for cases where the Java APIs simply are not suitable.