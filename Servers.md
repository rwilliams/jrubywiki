# Server Options for JRuby Applications

## Standard Java Servers

Most people deploy JRuby applications using typical Java application servers like Tomcat or JBoss. When using these servers, the usual solution is [[Warbler|http://kenai.com/projects/warbler/pages/Home]]. Warbler bundles up any [[Rack|http://rack.rubyforge.org/doc/]]-based application into a Java "Web Application aRchive" (WAR) file, which can then be deployed on any web application server.

The following is a list of servers known to work with JRuby. It is not meant to be exhaustive.

* Tomcat
* JBoss
* GlassFish
* WebLogic
* WebSphere

## Embedded/Micro Servers

The usual Ruby way to run a server is to launch a small command-line server pointing at the application directory. Some of the standard Ruby servers work with JRuby, and an additional set of servers have been built specifically for JRuby by either wrapping an existing server or by simply wrapping Java libraries.

The following is a list of embedded/micro/commandline servers for JRuby. If you don't see your favorite here, add it!

### Actively-developed Servers

* [[Trinidad|https://github.com/calavera/trinidad/wiki]] - A wrapper around [[Tomcat|http://tomcat.apache.org/]].
* [[Mizuno|https://github.com/matadon/mizuno]] - A wrapper around [[Jetty|http://jetty.codehaus.org/jetty/]].
* [[Kirk|https://github.com/strobecorp/kirk]] - Another Jetty wrapper, but less "micro" with redeploy and multiple appsupport.
* [[Aspen|https://github.com/kevwil/aspen]] - A server based on [[Netty|http://www.jboss.org/netty]], a Java NIO framework.

### Deprecated Servers

* Mongrel - The classic Ruby server, no longer maintained.
* GlassFish gem - A gem-borne embedded version of GlassFish, now no longer maintained by Oracle.

## Application Hosting Services

There are also several cloud and hosting services that either directly support JRuby or implicitly support it by having general Java webapp support.

The following services are known to work. Other webapp services *should* generally support JRuby applications (via Warbler) as well.

* [[Engine Yard|http://www.engineyard.com/]] - Direct support, in private beta now.
* [[Google AppEngine|http://code.google.com/p/appengine-jruby/wiki/GettingStarted]] - Indirect support, but a community exists around JRuby deployment
* [[Amazon Elastic BeanStalk|http://blog.headius.com/2011/01/jruby-on-rails-on-amazon-elastic.html]] - An elastic Java webapp cloud from Amazon based on Tomcat.