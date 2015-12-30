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

Using `rbenv` and `ruby-build` you can install `jruby-master+graal-dev`. Like the nightly builds, this includes both JRuby and the Graal VM so you don't need a separate JVM.

```
$ rbenv install jruby-master+graal-dev
$ rbenv shell jruby-master+graal-dev
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