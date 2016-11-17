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

JRuby+Truffle does not support RubyGems fully yet, therefore the `bundler-workarounds` file should be required to make it work. The file is part of JRuby+Truffle and can be always required.

* Installing a gem: `JRUBY_OPTS='-X+T' RUBYOPT='-r bundler-workarounds' gem install rake`
* Running bundler: `JRUBY_OPTS='-X+T' RUBYOPT='-r bundler-workarounds' bundle install`
* Executing in bundle: `JRUBY_OPTS='-X+T' RUBYOPT='-r bundler-workarounds' bundle exec rake db:create db:migrate`

The environment variables may also be exported for convenience `export JRUBY_OPTS='-X+T' RUBYOPT='-r bundler-workarounds` to be able to run just `bundle install`.

There is also a tool (`jruby-truffle-tool`) which helps to manage patches when needed, see its [documentation](https://github.com/jruby/jruby/blob/truffle-head/lib/ruby/truffle/jruby-truffle-tool/README.md).