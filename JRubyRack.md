
**NOTE:** contents of this page are being moved to **[[https://github.com/jruby/jruby-rack/wiki]]**.


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
