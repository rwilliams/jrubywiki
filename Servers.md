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

* [[Trinidad|https://github.com/trinidad/trinidad]] - A wrapper around [[Tomcat|http://tomcat.apache.org/]].
* [[Mizuno|https://github.com/matadon/mizuno]] - A wrapper around [[Jetty|http://jetty.codehaus.org/jetty/]].
* [[TorqueBox|http://torquebox.org]] - A server based on [[JBoss AS|http://www.jboss.org/jbossas]].
* [[Puma|http://puma.io]] - A server written in Ruby, wraps the Ragel parser (from Mongrel). 

### Deprecated Servers

* [[Kirk|https://github.com/strobecorp/kirk]] - Another Jetty wrapper, but less "micro" with redeploy and multiple appsupport.
* [[Aspen|https://github.com/kevwil/aspen]] - A server based on [[Netty|http://www.jboss.org/netty]], a Java NIO framework.
* Mongrel - The classic Ruby server, no longer maintained.
* GlassFish gem - A gem-borne embedded version of GlassFish, now no longer maintained by Oracle.

## Application Hosting Services

There are also several cloud and hosting services that either directly support JRuby or implicitly support it by having general Java webapp support.

The following services are known to work. Other webapp services *should* generally support JRuby applications (via Warbler) as well.

* [[Heroku|http://www.heroku.com]] - third-party support through build packs: [[github.com/carlhoerberg/heroku-buildpack-jruby|https://github.com/carlhoerberg/heroku-buildpack-jruby]]
* [[Engine Yard|http://www.engineyard.com/]] - Direct support, generally available now.
* [[Google AppEngine|http://code.google.com/p/appengine-jruby/wiki/GettingStarted]] - Indirect support, but a community exists around JRuby deployment
* [[Amazon Elastic BeanStalk|http://blog.headius.com/2011/01/jruby-on-rails-on-amazon-elastic.html]] - An elastic Java webapp cloud from Amazon based on Tomcat.
* [[OpenShift by Red Hat|http://www.openshift.com]] - Several possibilities
    * [[OpenShifter|http://blog.marekjelen.cz/article/openshifter-jruby-for-openshift]] - CLI tool to simplify the deployment of JRuby applications to OpenShift using JBoss application server
    * [[OpenShift-JRuby|https://github.com/marekjelen/openshift-jruby]] - Template for running JRuby application on OpenShift without an application server.
    * Deploying pre-build .war file to JBoss