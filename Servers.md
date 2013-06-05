# Server Options for JRuby Applications

## Standard Java Servers

Most people deploy JRuby applications using typical Java application servers like Tomcat or JBoss. When using these servers, the usual solution is [[Warbler|http://kenai.com/projects/warbler/pages/Home]]. Warbler bundles up any [[Rack|http://rack.rubyforge.org/doc/]]-based application into a Java "Web Application aRchive" (WAR) file, which can then be deployed on any web application server.

The following is a list of servers known to work with JRuby. It is not meant to be exhaustive.

* Tomcat
* JBoss
* GlassFish
* WebLogic
* WebSphere

## Other Servers

* [[Vert.x|http://vertx.io/]] - An asynchronous application platform on the JVM for writing high performance web enabled applications. Vert.x is a polyglot platform and lets you write your applications in multiple JVM languages including Ruby (JRuby). 

## Embedded/Micro Servers

The usual Ruby way to run a server is to launch a small command-line server pointing at the application directory. Some of the standard Ruby servers work with JRuby, and an additional set of servers have been built specifically for JRuby by either wrapping an existing server or by simply wrapping Java libraries.

The following is a list of embedded/micro/commandline servers for JRuby. If you don't see your favorite here, add it!

### Actively-developed Servers

* [[Trinidad|https://github.com/trinidad/trinidad]] - A wrapper around [[Tomcat|http://tomcat.apache.org/]].
* [[Mizuno|https://github.com/matadon/mizuno]] - A wrapper around [[Jetty|http://jetty.codehaus.org/jetty/]].
* [[Fishwife|https://github.com/dekellum/fishwife#readme]] - Server based on Jetty 7.x or 9.x
* [[TorqueBox|http://torquebox.org]] - A server based on [[JBoss AS|http://www.jboss.org/jbossas]].
* [[TorqueBox Lite|https://github.com/torquebox/torquebox-lite]] - A smaller, web-only version of TorqueBox.
* [[Puma|http://puma.io]] - A server written in Ruby, wraps the Ragel parser (from Mongrel). 
* [[Jubilee|https://github.com/isaiah/jubilee]] - A server based on [[Vertx|http://vertx.io]]

### Other Servers. Young Projects

* [[Thick|https://github.com/marekjelen/thick]] - A server based on [[Netty|http://www.netty.io]]

### Deprecated Servers

* [[Kirk|https://github.com/strobecorp/kirk]] - Another Jetty wrapper, but less "micro" with redeploy and multiple appsupport.
* [[Aspen|https://github.com/kevwil/aspen]] - A server based on [[Netty|http://www.jboss.org/netty]], a Java NIO framework.
* Mongrel - The classic Ruby server, no longer maintained.
* GlassFish gem - A gem-borne embedded version of GlassFish, now no longer maintained by Oracle.

## Application Hosting Services

There are also several cloud and hosting services that either directly support JRuby or implicitly support it by having general Java webapp support.

The following services are known to work. Other webapp services *should* generally support JRuby applications (via Warbler) as well.

* [[Heroku|http://www.heroku.com]] - Direct support.
* [[Engine Yard|http://www.engineyard.com/]] - Direct support, generally available now.
* [[Shelly Cloud|https://shellycloud.com]] - Direct support.
* [[Google AppEngine|http://code.google.com/p/appengine-jruby/wiki/GettingStarted]] - Indirect support, but a community exists around JRuby deployment
* [[Amazon Elastic BeanStalk|http://blog.headius.com/2011/01/jruby-on-rails-on-amazon-elastic.html]] - An elastic Java webapp cloud from Amazon based on Tomcat.
* [[OpenShift by Red Hat|http://www.openshift.com]] - Several possibilities
    * [[OpenShifter|http://blog.marekjelen.cz/article/openshifter-jruby-for-openshift]] - CLI tool to simplify the deployment of JRuby applications to OpenShift using JBoss application server
    * [[OpenShift-JRuby|https://github.com/marekjelen/openshift-jruby]] - Template for running JRuby application on OpenShift without an application server.
    * Deploying pre-build .war file to JBoss
* [[BitNami|http://bitnami.org/stack/jrubystack]] - Provides free Amazon Cloud Images for JRuby Stack which include MySQL, Tomcat, Java, JRuby and the commonly used gems (including warbler). BitNami Cloud Hosting is an optional commercial service with a free developer plan that provides scheduled backups, monitoring and customization for JRuby projects and dozens of other open source app and frameworks.