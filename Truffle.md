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

Developer documentation for JRuby+Truffle is at http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/doc/. This includes project outline, how to use Truffle and how to use Truffle-specific functionality.

This wiki page includes more informal installation instructions, status information and FAQs.

## Installation

### Nightly Builds

You can download a single tarball for Mac or Linux that includes JRuby and the GraalVM - so you don't even need a JVM installed on your system.

* http://lafo.ssw.uni-linz.ac.at/graalvm/jruby-dist-master+graal-macosx-x86_64-bin.tar.gz
* http://lafo.ssw.uni-linz.ac.at/graalvm/jruby-dist-master+graal-linux-x86_64-bin.tar.gz

```
$ wget http://lafo.ssw.uni-linz.ac.at/graalvm/jruby-dist-master+graal-macosx-x86_64-bin.tar.gz
$ tar -zxf jruby-dist-master+graal-macosx-x86_64-bin.tar.gz
$ jruby-master/bin/jruby -X+T -e 'puts Truffle.graal?'
true
```

### rbenv

> Outdated, please use rvm temporary until https://github.com/sstephenson/ruby-build/pull/807 is merged.

Using `rbenv` and `ruby-build` you can install `jruby-9.0.0.0+graal-dev`. Like the nightly builds, this includes both JRuby and the Graal VM so you don't need a separate JVM.

```
$ rbenv install jruby-9.0.0.0+graal-dev
$ rbenv shell jruby-9.0.0.0+graal-dev
$ ruby -X+T -e 'puts Truffle.graal?'
true
```

### rvm

Use `rvm` you can mount the nightly builds listed above. For example, on the Mac:

```
$ rvm mount -r http://lafo.ssw.uni-linz.ac.at/graalvm/jruby-dist-master+graal-macosx-x86_64-bin.tar.gz -n jruby-dev-graal
$ rvm use jruby-dev-graal
$ ruby -X+T -e 'puts Truffle.graal?'
true
```

### From Source

If you build JRuby from source you will also get JRuby+Truffle. You will need to download a release of Graal, and use this instead of your system Java, in order to get high performance.

* http://lafo.ssw.uni-linz.ac.at/graalvm/openjdk-8-graalvm-b132-linux-x86_64-0.7.tar.gz
* http://lafo.ssw.uni-linz.ac.at/graalvm/openjdk-8-graalvm-b132-macosx-x86_64-0.7.tar.gz

```
$ ./mvnw
$ JAVACMD=graalvm-jdk1.8.0/bin/java bin/jruby -X+T -e 'puts Truffle.graal?'
```

#### Latest Graal

You can also build against the latest version of Truffle and Graal if you use the `truffle-head` branch. Here you will need to build Graal for yourself.

https://wiki.openjdk.java.net/display/Graal/Instructions

```
$ hg clone https://bitbucket.org/allr/mx
$ export PATH=$PWD/mx:$PATH
$ mkdir graal
$ cd graal
$ mx sclone http://hg.openjdk.java.net/graal/graal-compiler
$ cd graal-compiler
$ mx build
```

```
$ JAVACMD=graal/jvmci/jdk1.8.0_51/product/bin/java bin/jruby -X+T -e 'puts Truffle.graal?'
```

### Demonstrating Truffle

However you install JRuby+Truffle, you may want to immediately demonstrate to yourself that Truffle is actually doing something. Put this program into `test.rb`:

```ruby
loop do
  14 + 2
end
```

Then run JRuby+Truffle (`jruby-master/bin/jruby` if via the nightly or `ruby` if via `rbenv`), with the `-X+T` option to use Truffle, and we'll also use the `-J-G:+TraceTruffleCompilation` to ask Truffle to tell us when it compiles something.


```
$ jruby-master/bin/jruby -X+T -J-G:+TraceTruffleCompilation test.rb
[truffle] opt done         BasicObject#equal?(core):core: BasicObject#equal? <opt>     |ASTSize       8/    8 |Time   408( 307+100 )ms |DirectCallNodes I    0/D    0 |GraalNodes    64/  137 |CodeSize          402 |Source core: BasicObject#equal? 
[truffle] opt done         block in loop:test.rb:1 <opt> <split-0-U>                |ASTSize      13/   20 |Time   256( 254+3   )ms |DirectCallNodes I    1/D    0 |GraalNodes    18/    3 |CodeSize           66 |Source   test.rb:1
```

Here you can see that Truffle has decided to compile the body of that loop to machine code - just 66 bytes of machine code in all. Along the way, it also decided to compile `BasicObject#equal?` - this is because that method is used enough times while we load the core library for the compiler to realise that it is hot and also compile it.

### Trying a gem or an application on Truffle

JRuby+Truffle does not support RubyGems yet, therefore there is a small gem `jruby+truffle_runner` to make testing of gems and applications on JRuby+Truffle easy. After the tool is installed just two commands have to be executed in the gems's or the application's directory: `jruby+truffle setup`, `jruby+truffle run a_file.rb`.

Please see [the documentation](https://github.com/jruby/jruby/blob/master/tool/truffle/jruby_truffle_runner/README.md) of `jruby+truffle_runner` tool.

## Current Status

### RubySpec

We track completeness against the suite of specs formerly known as RubySpec. These generated reports show overall completeness and work outstanding. We support over 92% of the language and over 89% of the core library. Work on the standard library is more limited.

* [Language Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-language-report/html/)
* [Core Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-core-report/html/)
* [Library Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-library-report/html/)

## Developing JRuby+Truffle

There are instructions for how to setup a working [Eclipse](Using-Eclipse-with-JRuby-Truffle) environment.  
A few recommendations are also available in [Developing with JRuby+Truffle](Developing-with-JRuby-Truffle).

## FAQ

### What is Truffle?

Truffle is a Java framework for writing AST interpreters. To implement a language using Truffle you write an AST for your language and add methods to interpret - perform the action of - each node.

Truffle also has the concept of specialisation. In most AST interpreters the nodes are megamorphic - they handle all possible types and other possible conditions. In Truffle you write several different nodes for the same semantic action but for different types and conditions. As runtime conditions change, you switch which nodes you are using. After the program has warmed up you should end up with an AST that is precisely tailored for the types and conditions that you are actually using. If these conditions change, you can just switch nodes again.

For more information Truffle see the [publications](https://wiki.openjdk.java.net/display/Graal/Publications+and+Presentations), [API documentation](http://lafo.ssw.uni-linz.ac.at/javadoc/graalvm/all/com/oracle/truffle/api/package-summary.html), and [FAQ](https://wiki.openjdk.java.net/display/Graal/Truffle+FAQ+and+Guidelines).

### What is Graal?

Graal is a new implementation of a JIT compiler in the OpenJDK Java Virtual Machine. Unlike the current compilers, Graal is written in Java, and exposes a Java API to the running program. This means that instead of emitting bytecode a JVM language can directly control the compiler. However this is complicated, so normally Truffle uses Graal on your behalf to *partially evaluate* your AST interpreter into machine code.

For more information about Graal see the [publications](https://wiki.openjdk.java.net/display/Graal/Publications+and+Presentations) and [API documentation](http://lafo.ssw.uni-linz.ac.at/javadoc/graalvm/all/index.html).

### Why is the Truffle backend slow on a standard JVM?

When running on a standard JVM, Truffle runs about as fast as the JRuby AST interpreter. By default JRuby compiles to bytecode, which is faster than interpreting an AST, so you may not be used to the performance of a simple AST interpreter and it may seem unusually slow.

Eventually the expected way to run Truffle programs will be using the Graal compiler, so the performance on a standard JVM will not be as important.

### Why is the Truffle backend faster on Graal?

When running on a VM with the Graal compiler, Truffle can use the API exposed by Graal. Truffle gets the IR representation of all of the AST interpreter methods involved in running your Ruby method, combines them into something like a single Java method, optimises them together, and emits a single machine code function. On Graal, Truffle also provides wrappers for JVM functionality not normally available to Java applications such as code deoptimization. The Truffle backend uses this to provide dramatically simpler and faster implementations of Ruby semantics compared to the current implementation in JRuby.

### Where did this code come from?

[Chris Seaton](https://github.com/chrisseaton) wrote an implementation of Ruby on Truffle and Graal as part of an internship at Oracle Labs in the first half of 2013. The code was merged into JRuby in early 2014. Benoit Daloze and Kevin Menard joined as researchers in the second half of 2014. Since then we have also accepted contributions from people outside Oracle Labs.

### Who do I ask about the Truffle backend?

Ask about Truffle in the `#jruby` Freenode IRC room. We'll get notified if you mention *truffle* so your question won't be missed.

### How do I know if I’m using a Graal VM?

`defined? Truffle` will tell you if you are running with Truffle, and `Truffle.graal?` will tell you if you are also running with the Graal dynamic compiler.

### Why doesn’t the Truffle backend work for my application or gem?

The Truffle backend implements almost all of the language but only about half the core library.

### Why doesn’t the Truffle backend perform well for my benchmark?

Benchmarks that we haven’t looked at yet are likely to require new code paths to be specialized. Currently we’ve added specialisation for the code paths in the benchmarks and applications that we’ve been using. Adding them is generally not complicated and over time we will have specialisations to cover a broad range of applications.

Also, check that you are using a Graal VM (see above).

### How is this related to `invokedynamic`?

The Truffle backend doesn’t use `invokedynamic`, as it doesn't emit bytecode. However it does have an optimising method dispatch mechanism that achieves a similar result - see the [method dispatch nodes](https://github.com/jruby/jruby/tree/master/truffle/src/main/java/org/jruby/truffle/nodes/dispatch).

### How is this related to the JRuby IR?

The Truffle backend and the JRuby IR are not related and take very different approaches. They are mutually exclusive backends - you can’t use both of them at the same time.

### Why isn't JRuby switching to Truffle now?

Truffle is a research project, there is no expected release date, and it does not perform well on a standard JVM. The JRuby+Truffle team aren't recommending that JRuby switch to Truffle. IR is what you want to be using today. Truffle is what you might want to run in the future, but not yet.