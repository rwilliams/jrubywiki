JRuby-Rack is a lightweight adapter for the Java servlet environment that allows any Rack-based application to run unmodified in a Java servlet container. JRuby-Rack supports Rails, Merb, as well as any Rack-compatible Ruby web framework.

For more information on Rack, visit [[http://rack.rubyforge.org]].

Download
--------

JRuby-Rack 0.9 is the first public release. Download it:

[[http://repository.codehaus.org/org/jruby/rack/jruby-rack/0.9/jruby-rack-0.9.jar]]

JRuby-Rack is also bundled with [http://caldersphere.rubyforge.org/warbler Warbler 0.9.9], which is available as a Ruby gem (<tt>jruby -S gem install warbler</tt>).

Features
--------

Servlet Filter
--------------

JRuby-Rack's main mode of operation is as a servlet filter. This allows requests for static content to pass through and be served by the
application server. Dynamic requests only happen for URLs that don't have a corresponding file, much like many Ruby applications expect.

Goldspike-compatible Servlet
----------------------------

JRuby-Rack includes a stub RailsServlet and recognizes many of Goldspike's context parameters (e.g., pool size configuration), making it
interchangeable with Goldspike, for convenience of migration. One caveat is that static content is served by Rack, which requires acquisition of
a JRuby runtime. Your throughput with static files will be much lower than when JRuby-Rack is configured as a servlet filter. You have been
warned!

Servlet environment integration
-------------------------------

* Servlet context is accessible to any application both through the global variable $servlet_context and the Rack environment variable java.servlet_context.
* Servlet request object is available in the Rack environment via the key java.servlet_request.
* Servlet request attributes are passed through to the Rack environment.
* Rack environment variables and headers can be overridden by servlet request attributes.
* Java servlet sessions are used as the default session store for both Rails and Merb. Session attributes with String keys and String, numeric, boolean, or java object values are automatically copied to the servlet session for you.

JRuby Runtime Management
------------------------

JRuby runtime management and pooling is done automatically by the framework. In the case of Rails, runtimes are pooled. For Merb and other Rack
applications, a single runtime is created and shared for every request.

Feedback
--------

For simplicity, I haven't set up any separate mailing lists or bug trackers for jruby-rack at this time. Please use the [http://xircles.codehaus.org/lists/user@jruby.codehaus.org jruby-user mailing list] and the [http://jira.codehaus.org/browse/JRUBY JRuby JIRA issue tracker] for feedback.

Source
------

The source is currently available from the jruby-contrib subversion repository on codehaus:

http://svn.codehaus.org/jruby-contrib/trunk/rack

Building
--------

Ensure you have JRuby with the buildr and rack gems installed.

  jruby -S gem install buildr rack

Checkout the JRuby Rack code and cd to that directory

  svn co http://svn.codehaus.org/jruby-contrib/trunk/rack
  cd rack

Resolve dependencies, compile the code, and build the jar file.

  jruby -S buildr package

The generated jar should be found in target/jruby-rack-*.jar.

**NOTE**: The JRuby 1.1.3 release, which bundles Rubygems 1.2, [http://rubyforge.org/tracker/index.php?func=detail&aid=21056&group_id=126&atid=575 has some problems with installing Buildr]. To that end I've prepared a standalone buildr package with JRuby 1.1.2. [http://caldersphere.net/jruby-rack/jruby-buildr.tar.gz Download it], add the jruby-buildr directory to your path, and use the jruby-buildr/buildr script instead of "jruby -S buildr" above.
