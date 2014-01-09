The Truffle runtime of JRuby is an experimental implementation of an interpreter for JRuby using the Truffle AST interpreting framework and the Graal compiler. It’s a potential alternative to the current AST interpreter, the bytecode backend, and the new IR. The goal is to be both significantly faster and simpler than other high performance implementations of Ruby.

This wiki page is a collection of notes about the runtime. For more general background information see the [announcement blog post](http://blog.jruby.org/2014/01/truffle_graal_high_performance_backend/), and the FAQ below.

Most of the work for the truffle runtime is in [core/src/main/java/org/jruby/truffle](https://github.com/jruby/jruby/tree/master/core/src/main/java/org/jruby/truffle).

Current Status
===========

At the moment the Truffle backend is not much more than the Oracle Labs implementation of Ruby, hosted within JRuby. The runtimes, object representation and core libraries are still very separate, but the goal is to integrate them.

The Truffle backed is unlikely to work for arbitrary code without having to implement a few features here and there. Even if it does work, performance is unlikely to be good for arbitrary code without having to implement additional specialisations. However we can run several micro-benchmarks and RubySpec, where we pass about half the language specs.

Compiling
========

The Truffle backend is integrated into the normal JRuby build system.

    mvn package

Running Truffle
============

To enable the Truffle backend use the `-X+T` option. `-X+T` also turns off loading the Ruby kernel and implies `--disable-gems`.

     bin/jruby -X+T

Running With Graal
===============


The Truffle backend will run on any Java 7+ JVM, but it will only JIT and optimize when running on top of a Graal-enabled build of OpenJDK.

To run Truffle on top of the Graal compiler, you need an installation of the Graal VM. You can download [Graal builds for 64-bit Linux and Mac](http://lafo.ssw.uni-linz.ac.at/graalvm/), or see below for how to build your own.

The binary releases of Graal look like a normal JVM install. Put bin/ in PATH and set JAVA_HOME and JRuby will pick up that VM build.

To tell JRuby to use the non-standard `mx.sh` command in local Graal builds, set the `JAVACMD` environment variable. You should also explicitly set `-server` and `-d64`, and to check that you really are using Graal, we set `-Xtruffle.printRuntime=true` so that it prints the name of the Truffle runtime as it starts. You should see ‘Graal’.

     JAVACMD=path/to/graal/jdk1.7.0_45/product/bin/java bin/jruby -J-server -J-d64 -X+T -Xtruffle.printRuntime=true

Building Graal
===========

Graal is a fork of OpenJDK. Compiling it isn’t too complicated, but it isn’t part of the JRuby build system so you’ll have to do it separately.

You will need a system and C/C++ compiler toolchain supported by OpenJDK, as well as Mercurial, Ant and Python 2.7.

     hg clone http://hg.openjdk.java.net/graal/graal
     cd graal
     ./mx.sh --vm server-nograal --vmbuild product build
     ./mx.sh --vm server --vmbuild product build

This will give you a Java VM executable with Graal in `graal/jdk1.7.0_45/product/bin/java`. 

If you are on OS X Mavericks you will need to set these variables:

    export COMPILER_WARNINGS_FATAL=false
    export USE_PRECOMPILED_HEADER=0
    export USE_CLANG=true
    export LFLAGS="-Xlinker -lstdc++"

Running RubySpec
===============

The Truffle backend currently uses a separate copy of RubySpec and tags. It passes about half the language specs. Instructions for running RubySpec are in [spec/truffle/README](https://github.com/jruby/jruby/tree/truffle/spec/truffle/README).

As with the rest of implementation, this should be merged with the normal set of RubySpecs over time.

Running Benchmarks
================

We have a couple of micro-benchmarks that we know work in bench/truffle. They are very limited, are focused on peak performance, don’t include time for warmup and don’t have any rigorous statistical methodology behind them, so don’t expect to draw too many conclusions. For example, to run Mandelbrot:

     cd bench/truffle
     ../../bin/jruby -J-server -J-d64 -X+T -Xtruffle.printRuntime=true harness.rb -s 120 mandelbrot.rb
     JAVACMD=path/to/graal/jdk1.7.0_45/product/bin/java ../../bin/jruby -J-server -J-d64 -Xtruffle -Xtruffle.printRuntime=true harness.rb -s 120 mandelbrot.rb

You should see something very roughly like an 10-15x increase in the score compared to `invokedynamic` - a 10-15x speedup, and more if you compare against JRuby without `invokedynamic`.

Truffle Options
===========

     -X+T

Use Truffle as the ‘compile mode’. Turns off loading the Ruby kernel and implies `--disable-gems`.

     -Xtruffle.printRuntime=true

Print the name of the Truffle runtime that you are using. ‘Default’ means Truffle running as a normal Java library - which will be about as slow as the normal JRuby AST interpreter. ‘Graal’ means that you are using Graal VM to compile the Truffle interpreter to native code, and should be significantly faster.

FAQ
===

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

The Truffle backend doesn’t use `invokedynamic`, as it doesn't emit bytecode. However it does have an optimising method dispatch mechanism that achieves a similar result - see the [method dispatch nodes](https://github.com/jruby/jruby/tree/truffle/core/src/main/java/org/jruby/truffle/nodes/call).

**How is this related to the new IR?**

The Truffle backend and the new IR are not related and take very different approaches. They are mutually exclusive backends - you can’t use both of them at the same time.

**Why are there are assertion failures while processing annotations during compilation under IntelliJ IDEA?**

The assertion failures are a bug in Truffle - sorry about them. You should be able to pass `-J-da` to `javac` in preferences, but that doesn’t seem to fix the problem. Compiling in Maven works fine.