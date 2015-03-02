# JRuby+Truffle - a High-Performance Truffle Backend for JRuby

The Truffle runtime of JRuby is an experimental implementation of an interpreter
for JRuby using the Truffle AST interpreting framework and the Graal compiler.
It’s an alternative to the IR interpreter and bytecode compiler. The goal is to
be significantly faster, simpler and to have more functionality than other
implementations of Ruby.

JRuby+Truffle is a project of [Oracle Labs](https://labs.oracle.com) and
academic collaborators at the [Institut für Systemsoftware at Johannes Kepler
University Linz](http://ssw.jku.at).

## Documentation

Documentation for JRuby+Truffle is at http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/doc/. This includes project outline, how to use Truffle and how to use Truffle-specific functionality.

This wiki page includes status information and FAQs.

## Current Status

## RubySpec

We track completeness against the suite of specs formerly known as RubySpec. These generated reports show overall completeness and work outstanding.

* [Language Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-language-report/html/)
* [Core Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-core-report/html/)
* [Stdlib Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-rubysl-report/html/)

## FAQ

**What is Truffle?**

Truffle is a Java framework for writing AST interpreters. To implement a language using Truffle you write an AST for your language and add methods to interpret - perform the action of - each node.

Truffle also has the concept of specialisation. In most AST interpreters the nodes are megamorphic - they handle all possible types and other possible conditions. In Truffle you write several different nodes for the same semantic action but for different types and conditions. As runtime conditions change, you switch which nodes you are using. After the program has warmed up you should end up with an AST that is precisely tailored for the types and conditions that you are actually using. If these conditions change, you can just switch nodes again.

For more information Truffle see the [publications](https://wiki.openjdk.java.net/display/Graal/Publications+and+Presentations), [API documentation](http://lafo.ssw.uni-linz.ac.at/javadoc/graalvm/all/com/oracle/truffle/api/package-summary.html), and [FAQ](https://wiki.openjdk.java.net/display/Graal/Truffle+FAQ+and+Guidelines).

**What is Graal?**

Graal is a new implementation of a JIT compiler in the OpenJDK Java Virtual Machine. Unlike the current compilers, Graal is written in Java, and exposes a Java API to the running program. This means that instead of emitting bytecode a JVM language can directly control the compiler. However this is complicated, so normally Truffle uses Graal on your behalf.

For more information about Graal see the [publications](https://wiki.openjdk.java.net/display/Graal/Publications+and+Presentations) and [API documentation](http://lafo.ssw.uni-linz.ac.at/javadoc/graalvm/all/index.html).

**Why is the Truffle backend slow on a standard JVM?**

When running on a standard JVM, Truffle runs about as fast as the JRuby AST interpreter. By default JRuby compiles to bytecode, which is faster than interpreting an AST, so you may not be used to the performance of a simple AST interpreter and it may seem unusually slow.

Eventually the expected way to run Truffle programs will be using the Graal compiler, so the performance on a standard JVM will not be as important.

**Why is the Truffle backend faster on Graal?**

When running on a VM with the Graal compiler, Truffle can use the API exposed by Graal. Truffle gets the IR representation of all of the AST interpreter methods involved in running your Ruby method, combines them into something like a single Java method, optimises them together, and emits a single machine code function. On Graal, Truffle also provides wrappers for JVM functionality not normally available to Java applications such as code deoptimization. The Truffle backend uses this to provide dramatically simpler and faster implementations of Ruby semantics compared to the current implementation in JRuby.

**Where did this code come from?**

[Chris Seaton](https://github.com/chrisseaton) wrote an implementation of Ruby on Truffle and Graal as part of an internship at Oracle Labs in the first half of 2013. The code in the `org.jruby.truffle` package is that open source code merged into JRuby.

**Who do I ask about the Truffle backend?**

[Chris Seaton](https://github.com/chrisseaton) is the point of contact, or discuss on the JRuby developer’s mailing list. General Graal and Truffle issues can be discussed on the [Graal mailing list](http://mail.openjdk.java.net/mailman/listinfo/graal-dev).

**How do I know if I’m using a Graal VM?**

Use `-Xtruffle.printRuntime=true`. This should print the name of the Truffle runtime that you are using. ‘Default’ means Truffle running as a normal Java library - which will be about as slow as the normal JRuby AST interpreter. ‘Graal’ means that you are using Graal VM to compile the Truffle interpreter to native code, and should be significantly faster.

**Why doesn’t the Truffle backend work for my application or gem?**

The Truffle backend is not yet mature and doesn’t support all of Ruby. It passes about half of the RubySpec language specs, so is unlikely to work for arbitrary code.

**Why doesn’t the Truffle backend perform well for my benchmark?**

Benchmarks that we haven’t looked at yet are likely to require new code paths to be specialized. Currently we’ve added specialisation for the code paths in the benchmarks and applications that we’ve been using. Adding them is generally not complicated and over time we will have specialisations to cover a broad range of applications.

Also, check that you are using a Graal VM (see above).

**How is this related to `invokedynamic`?**

The Truffle backend doesn’t use `invokedynamic`, as it doesn't emit bytecode. However it does have an optimising method dispatch mechanism that achieves a similar result - see the [method dispatch nodes](https://github.com/jruby/jruby/tree/master/core/src/main/java/org/jruby/truffle/nodes/call).

**How is this related to the new IR?**

The Truffle backend and the new IR are not related and take very different approaches. They are mutually exclusive backends - you can’t use both of them at the same time.

**Why are there are assertion failures while processing annotations during compilation under IntelliJ IDEA?**

The assertion failures are a bug in Truffle - sorry about them. You should be able to pass `-J-da` to `javac` in preferences, but that doesn’t seem to fix the problem. Compiling in Maven works fine.