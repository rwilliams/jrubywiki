Roadmap
=======

JRuby 1.6 (end of life as of 1.6.8.1)
--------------------------

We are no longer maintaining this release and encourage all users to upgrade to 1.7.x.

JRuby 1.7 (current)
--------------------------

JRuby 1.7 is the current major version of JRuby.   This release was primarily about production-level support for Ruby 1.9.3.  A major improvement internally was supporting Java 7 invokedynamic.

JRuby 2.0 (future - also referred to as 9k - or 9000)
------------------

JRuby 2.0 will be the first major release where we have nontrivial breaking changes along with new features. It's our chance to clean things up that have lingered for years.  Plans at the moment is to start master cut over after JRuby 1.7.4 or JRuby 1.7.5 (this depends on confidence of maturity of 1.9.3 support after releasing 1.7.4).

The following list includes mostly proposed changes that still need to be discussed (several of these depend on conditions out of our control like adoption):

* ClassLoader-global JRuby runtime, to eliminate getRuntime() calls.
* Using IR runtime as our next-gen runtime
  * Incremental loading with intermediate representation persisted
* Removal of Ruby 1.8 support
* Addition of Ruby 2.0 support
* Possible removal of 1.9 support (depends on 2.0 uptake)
* Java 7+ as a hard requirement (depends on Java 7 adoption)
* Modularity for custom packaging (loose source/package coupling feature for android, GAE reduce-scope releases) 
