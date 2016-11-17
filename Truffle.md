# JRuby+Truffle - a High-Performance Implementation of Ruby using Truffle and Graal

The JRuby+Truffle runtime is an experimental implementation of Ruby
using the Truffle AST interpreting framework and the Graal compiler.
It’s an alternative to the JRuby IR interpreter and bytecode compiler.
The goal is to be significantly faster, simpler and
to have more functionality than other implementations of Ruby.

JRuby+Truffle is a project of [Oracle Labs](https://labs.oracle.com) and
academic collaborators at the [Institut für Systemsoftware at Johannes Kepler
University Linz](http://ssw.jku.at).

## Wiki Pages

* [[Downloading GraalVM]]
* [[Using Graal in JDK 9 EA Builds]]
* [[Building Graal]]
* [[JRuby and Truffle Interop]]
* [[Truffle FAQ]]

## Developer Documentation

Developer documentation for JRuby+Truffle is at https://github.com/jruby/jruby/blob/truffle-head/truffle/README.md, and some additional pages in this wiki:

* [[Developing with JRuby Truffle]]
* [[Using Eclipse with JRuby Truffle]]
* [Other resources](https://gist.github.com/smarr/d1f8f2101b5cc8e14e12)

## Workaround for RubyGems

JRuby+Truffle does not support RubyGems yet, therefore there is a small tool `jruby+truffle` to make testing of gems and applications on JRuby+Truffle easy. The tool is bundled in JRuby. To install dependencies run `jruby+truffle setup` in the gems's or the application's directory, then`jruby+truffle run a_file.rb` to try the gem or application.

Please see [the documentation](https://github.com/jruby/jruby/blob/master/lib/ruby/truffle/jruby%2Btruffle/README.md) of `jruby+truffle_runner` tool.