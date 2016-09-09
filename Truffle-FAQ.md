### What is Truffle?

Truffle is a Java framework for writing AST interpreters. To implement a language using Truffle you write an AST for your language and add methods to interpret - perform the action of - each node.

Truffle also has the concept of specialisation. In most AST interpreters the nodes are megamorphic - they handle all possible types and other possible conditions. In Truffle you write several different nodes for the same semantic action but for different types and conditions. As runtime conditions change, you switch which nodes you are using. After the program has warmed up you should end up with an AST that is precisely tailored for the types and conditions that you are actually using. If these conditions change, you can just switch nodes again.

For more information Truffle see the [publications](https://github.com/graalvm/graal-core/blob/master/docs/Publications.md).

### What is Graal?

Graal is a new implementation of a JIT compiler in the OpenJDK Java Virtual Machine. Unlike the current compilers, Graal is written in Java, and exposes a Java API to the running program. This means that instead of emitting bytecode a JVM language can directly control the compiler. However this is complicated, so normally Truffle uses Graal on your behalf to *partially evaluate* your AST interpreter into machine code.

For more information about Graal see the [publications](https://github.com/graalvm/graal-core/blob/master/docs/Publications.md).

### How do I get JRuby+Truffle?

The same way you would normally get JRuby, such as from http://jruby.org/download, `rvm`, `ruby-build`, or `ruby-install`, but without the Graal compiler it isn't very interesting so you will need that as well.

* [[Downloading GraalVM]]

Then you can run JRuby setting the `JAVACMD` environment variable and use Truffle by specifying `-X+T`.

```
JAVACMD=graalvm-0.16-re/bin/java jruby -X+T ....
```

### Why is the Truffle backend slow on a standard JVM?

When running on a standard JVM, Truffle runs about as fast as the JRuby AST interpreter. By default JRuby compiles to bytecode, which is faster than interpreting an AST, so you may not be used to the performance of a simple AST interpreter and it may seem unusually slow.

The expected way to run Truffle programs is using the Graal compiler, so the performance on a standard JVM is not very important.

### Why is the Truffle backend faster on Graal?

When running on a VM with the Graal compiler, Truffle can use the API exposed by Graal. Truffle gets the IR representation of all of the AST interpreter methods involved in running your Ruby method, combines them into something like a single Java method, optimises them together, and emits a single machine code function. On Graal, Truffle also provides wrappers for JVM functionality not normally available to Java applications such as code deoptimization. The Truffle backend uses this to provide dramatically simpler and faster implementations of Ruby semantics compared to the current implementation in JRuby.

### Where did this code come from?

[Chris Seaton](https://github.com/chrisseaton) wrote an implementation of Ruby on Truffle and Graal as part of an internship at Oracle Labs in the first half of 2013. The code was merged into JRuby in early 2014. Benoit Daloze and Kevin Menard joined as researchers in the second half of 2014, then Petr Chalupa in 2015 and Bradon Fish in 2016. Since then we have also accepted contributions from people outside Oracle Labs.

### Who do I ask about the Truffle backend?

Ask about Truffle in the `#jruby` Freenode IRC room. We'll get notified if you mention *truffle* so your question won't be missed. You can also email chris.seaton@oracle.com.

### How do I know if I’m using a VM that has Graal?

`defined? Truffle` will tell you if you are running with Truffle, and `Truffle::Graal.graal?` will tell you if you are also running with the Graal dynamic compiler.

### How can I see that Truffle and Graal are working?

Put this program into `test.rb`:

```ruby
loop do
  14 + 2
end
```

As well as the instructions for running with [GraalVM](Downloading GraalVM), we'll also use the `-J-G:+TraceTruffleCompilation` to ask Truffle to tell us when it compiles something.


```
$ JAVACMD=graalvm-0.16-re/bin/java jruby -J-G:+TraceTruffleCompilation -X+T test.rb
[truffle] opt done         + core <opt> <split-1947596f>                               |ASTSize       6/    6 |Time    95(  92+3   )ms |DirectCallNodes I    0/D    0 |GraalNodes    35/   28 |CodeSize          147 |Source           core 
[truffle] opt done         block in block in <main> /Users/chrisseaton/Documents/ruby/test.rb:1 <opt> <split-44b29496>|ASTSize      10/   16 |Time   124( 122+2   )ms |DirectCallNodes I    1/D    0 |GraalNodes    24/    3 |CodeSize           69 |Source /Users/chrisseaton/Documents/ruby/test.rb:1 
[truffle] opt done         truffle:/jruby-truffle/core/kernel.rb:331<OSR> <opt>        |ASTSize       8/   24 |Time    77(  74+4   )ms |DirectCallNodes I    2/D    0 |GraalNodes    89/  124 |CodeSize          341 |Source            n/a 
```

Here you can see that Truffle has decided to use Graal to compile the body of that loop to machine code - just 66 bytes of machine code in all. Along the way, it also decided to compile `BasicObject#equal?` - this is because that method is used enough times while we load the core library for the compiler to realise that it is hot and also compile it.

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