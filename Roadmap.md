Roadmap
=======

JRuby 1.6 (end-of-life as of 1.6.8.1)
--------------------------

We are no longer maintaining this release and encourage all users to upgrade to JRuby 9000.

JRuby 1.7
--------------------------

This release was primarily about production-level support for Ruby 1.9.3.  A major improvement internally was supporting Java 7 invokedynamic.

JRuby 9000 (current)
------------------

JRuby 9.0 is the current major version of JRuby. It implements (MRI) Ruby 2.2. JRuby 9000 was the first major release where we had nontrivial breaking changes along with new features. It was our chance to clean things up that had lingered for years. JRuby 9.0.0.0 was released July 22nd, 2015.

"The unusual version name came about as the team realized the next natural JRuby version would be either 1.8 or 2.0 and thus decided to avoid confusion with the Ruby MRI versions by using 9000." (ref: [infoq.com article summarizing @headius talk at Baruco](http://www.infoq.com/news/2013/09/jruby-9k))

Major features of JRuby 9000:

* Ruby 2.2 compatibility
* A new optimizing runtime based on a traditional compiler design
* New POSIX-friendly IO and Process
* Fully ported encoding/transcoding logic from MRI