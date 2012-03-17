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

## Runtime code optimization

JRuby currently has an intermediate representation (IR) that attempts to capture high-level Ruby semantics via instructions and operands.  This IR will be the basis of an updated JRuby VM.  While some optimizations are already in place including dead code elimination, method and block inlining (incomplete), there are lots of opportunities for improving on these and implementing additional optimizations.  A student interested in interpreters, compilers, virtual machines would work with the JRuby team to expand on the capabilities of this VM -- projects could include work on the interpreter, new performance optimizations, implementing new backends (ex: Dalvik).