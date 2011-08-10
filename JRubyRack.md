JRuby-Rack is a lightweight adapter for the Java servlet environment that allows any Rack-based application to run unmodified in a Java servlet container. JRuby-Rack supports Rails, Merb, as well as any Rack-compatible Ruby web framework.

For more information on Rack, visit [[http://rack.rubyforge.org]].

Download
--------


JRuby-Rack 1.0.9 is the latest version. Download it:

[[http://repository.codehaus.org/org/jruby/rack/jruby-rack/1.0.9/jruby-rack-1.0.9.jar]]

JRuby-Rack is also bundled with [http://caldersphere.rubyforge.org/warbler Warbler 0.9.9], which is available as a Ruby gem (`jruby -S gem install warbler`).

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

* Servlet context is accessible to any application both through the global variable `$servlet_context` and the Rack environment variable `java.servlet_context`.
* Servlet request object is available in the Rack environment via the key `java.servlet_request`.
* Servlet request attributes are passed through to the Rack environment.
* Rack environment variables and headers can be overridden by servlet request attributes.
* Java servlet sessions are used as the default session store for both Rails and Merb. Session attributes with String keys and String, numeric, boolean, or java object values are automatically copied to the servlet session for you.

JRuby Runtime Management
------------------------

JRuby runtime management and pooling is done automatically by the framework. In the case of Rails, runtimes are pooled. For Merb and other Rack
applications, a single runtime is created and shared for every request.

Feedback
--------

For simplicity, I haven't set up any separate mailing lists or bug trackers for jruby-rack at this time. Please use the [jruby-user mailing list](http://xircles.codehaus.org/lists/user@jruby.codehaus.org)  and the [JRuby JIRA issue tracker](http://jira.codehaus.org/browse/JRUBY) for feedback.

Source
------

The source is currently available from [[https://github.com/nicksieger/jruby-rack]] on github:

Building
--------

check instructions on [README](https://github.com/nicksieger/jruby-rack/blob/master/README.md) file
