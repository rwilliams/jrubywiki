## How to enable multi-threading in rails

To enable thread-safe (multi-threaded) mode in Rails when running with a warbler-generated WAR file:

* in config/environment.rb put the line ''config.threadsafe!''
* set min/max runtimes in warble.rb to 1/1 respectively (or just remove those lines, the default is already 1)

When running with the GlassFish gem, threadsafe mode will be detected automatically and GlassFish will use only a single JRuby instance. As a result only step 2 above is necessary.

## Code Best Practices for thread-safety

### Collection Thread-Safety

As of JRuby 1.2, most of the core collection types are not thread-safe. The overhead involved in ensuring that Array, Hash, String, and other types are thread-safe has been considered unreasonable given the vast majority of uses will not involve cross-thread modification. This means that you will need to handle thread-safety at a Ruby level when concurrently accessing and modifying most of the stateful core classes.

JRuby does, on the other hand, guarantee thread-safe instance variables, method tables, and IO streams.

These facts are subject to change in future versions of JRuby, but only after discussion with the community.

### In User Code

In general the simplest best practice for thread-safety is to never expect unsynchronized data stores to be shared across threads. Usually this is as simple as not using class variables or class instance variables to cache information, since most of the Rails objects a request encounters are only alive for the duration of a request.

### In Library Code

Libraries often are more difficult to make thread-safe, since they may have more objects shared across threads. As much as possible, mutable state should not be shared across threads. Where such sharing is unavoidable, you should use a Mutex around accesses to ensure that concurrent modifications do not corrupt data or damage a collection.

## Application Server Optimization/Configuration

TBD

## Related articles

* [Q/A: What Thread-safe Rails Means](http://blog.headius.com/2008/08/qa-what-thread-safe-rails-means.html)