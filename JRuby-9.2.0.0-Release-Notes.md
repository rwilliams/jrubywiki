
## JRuby 9.2.0.0

<!-- Ruby 2.3/2.4 features yet to be implemented: -->

### Notable changes since 9.1

<!-- NONE -->

#### Java Integration

* due a native (ruby) `Set` implementation sets now implement (are Java coercible) `java.util.Set`
* due a native (ruby) `SortedSet` implementation sets now implement (are Java coercible) `java.util.SortedSet`
* Java numbers now seamlessly work on operations where Ruby `Integer`/`Float` are expected 

  e.g. `[ java.lang.Integer.new(1), 2 ].pack('S')`

* the internal (previously deprecated) `JavaPackageModuleTemplate` has been removed use `JavaPackage` instead
* refactored and deprecated `Class#subclasses` (from *jruby/core_ext.rb*)
* moved `String#unseeded_hash` extension (from *require 'jruby'*) to *jruby/core_ext/string.rb*


### [Issues/Features Resolved in 9.2.0.0](https://github.com/jruby/jruby/issues?q=milestone%3A%22JRuby+9.2.0.0%22+is%3Aclosed)
