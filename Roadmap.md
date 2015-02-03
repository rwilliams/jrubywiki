Roadmap
=======

JRuby 1.6 (end of life as of 1.6.8.1)
--------------------------

We are no longer maintaining this release and encourage all users to upgrade to 1.7.x.

JRuby 1.7 (current)
--------------------------

JRuby 1.7 is the current major version of JRuby.   This release was primarily about production-level support for Ruby 1.9.3.  A major improvement internally was supporting Java 7 invokedynamic.

JRuby 9000
------------------

JRuby 9000 will be the first major release where we have nontrivial breaking changes along with new features. It's our chance to clean things up that have lingered for years. It will implement (MRI) Ruby 2.2. JRuby 9.0.0.0.pre1 was released Jan 20th, 2015. A final version is expected to be released Q1 2015.

"The unusual version name came about as the team realized the next natural JRuby version would be either 1.8 or 2.0 and thus decided to avoid confusion with the Ruby MRI versions by using 9000." (ref: [infoq.com article summarizing @headius talk at Baruco](http://www.infoq.com/news/2013/09/jruby-9k))

The following list includes mostly proposed changes that still need to be discussed (several of these depend on conditions out of our control like adoption):

* ClassLoader-global JRuby runtime, to eliminate getRuntime() calls.
* Using IR runtime as our next-gen runtime
  * Incremental loading with intermediate representation persisted
* Removal of Ruby 1.8 and 1.9 support
* Addition of Ruby 2.2 support
* Java 7+ as a hard requirement (depends on Java 7 adoption)
* Modularity for custom packaging (loose source/package coupling feature for android, GAE reduce-scope releases) 