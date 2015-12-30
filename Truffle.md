# JRuby+Truffle - a High-Performance Truffle Backend for JRuby

The Truffle runtime of JRuby is an experimental implementation of an interpreter
for JRuby using the Truffle AST interpreting framework and the Graal compiler.
It’s an alternative to the IR interpreter and bytecode compiler. The goal is to
be significantly faster, simpler and to have more functionality than other
implementations of Ruby.

JRuby+Truffle is a project of [Oracle Labs](https://labs.oracle.com) and
academic collaborators at the [Institut für Systemsoftware at Johannes Kepler
University Linz](http://ssw.jku.at).

## Wiki Pages

* [[Downloading GraalVM]]
* [[Using Graal in JDK-9 EA Builds]]
* [[Building Graal]]
* [[Truffle FAQ]]

## Other Documentation

Developer documentation for JRuby+Truffle is at http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/doc/. This includes project outline, how to use Truffle and how to use Truffle-specific functionality.

## Workaround for RubyGems

JRuby+Truffle does not support RubyGems yet, therefore there is a small gem `jruby+truffle_runner` to make testing of gems and applications on JRuby+Truffle easy. After the tool is installed just two commands have to be executed in the gems's or the application's directory: `jruby+truffle setup`, `jruby+truffle run a_file.rb`.

Please see [the documentation](https://github.com/jruby/jruby/blob/master/tool/truffle/jruby_truffle_runner/README.md) of `jruby+truffle_runner` tool.

## RubySpec

We track completeness against the suite of specs formerly known as RubySpec. These generated reports show overall completeness and work outstanding. We support 93% of the language and over 90% of the core library.

* [Language Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-language-report/html/)
* [Core Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-core-report/html/)
* [Library Specs Report](http://lafo.ssw.uni-linz.ac.at/graalvm/jruby/specs-library-report/html/)

## Developing JRuby+Truffle

* [[Using Eclipse with JRuby Truffle]]
* [[Developing with JRuby Truffle]]