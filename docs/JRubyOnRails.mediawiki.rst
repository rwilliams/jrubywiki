[[Home|&raquo; JRuby Project Wiki Home Page]]
<h1>JRuby on Rails</h1>
You can use JRuby with [http://www.rubyonrails.org/ Ruby on Rails]. JRuby gives Rails the power and functionality of the Java Platform,  providing it with:
* Excellent garbage collection for endless uptimes
* Hotspot profiled dynamic optimizations for great performance
* Access to the Java ecosphere for additional technology options
* Deployment to Java application servers for ubiquity

__TOC__
===Getting Started===
Get started with these tutorials:

* [http://deployingjruby.blogspot.com/2014/12/using-puma-with-rails-4-on-heroku.html Using Puma, Rails 4 and JRuby on Heroku] - Simple well explained tutorial (Up to date-ish. Written on December 4, 2014).
* Outdated(404): <strike>[http://blog.rubyrockers.com/2011/03/rails3-application-jruby Rails3 application with Jruby]</strike>
* Outdated(JRuby Version 1.3.1): <strike>[http://thenice.tumblr.com/post/133345213/deploying-a-rails-application-in-tomcat-with-jruby-a Deploying a Rails application in Tomcat with JRuby: A concise tutorial]</strike>
* Empty Wiki: <strike>[[JRuby on Rails with Spring|JRuby on Rails with Spring]] - zero to <code>.war</code> file with Spring and Maven.</strike>
* Outdated (Page says 'This book is obsolete') <strike>"Porting" the Depot Application To Jruby, [http://tompurl.com/2010/03/02/running-the-depot-application-with-jruby/ Parts 1] and [http://tompurl.com/2010/03/25/running-depot-on-tomcat/ 2]: These tutorials show you how to make the Depot application from "[http://www.pragprog.com/titles/rails3/agile-web-development-with-rails-third-edition Agile Web Development with Rails, Third Edition]" work with Jruby.</strike>

=== Deployment ===

==== This page is outdated, more up-to-date info on our [[Servers]] page ====

==== War File Deployment ====
If you don't use a traditional Ruby application server like [http://mongrel.rubyforge.org/ Mongrel], you can use a Java application server.  To deploy to a Java app server, you can use the tool <a href="https://github.com/jruby/warbler">Warbler</a> to bundle your Rails application in a Java Web Application Archive (<tt>.war</tt> file). (Warbler is a minimal, flexible, Ruby-like way to create a <tt>.war</tt> file.) Once you have a <tt>.war</tt> file, you can deploy to any Java app server using its war deployment mechanism.  

'''Some links to information on various Java appllication servers:'''
* [http://atalks.prokhorenko.us/2009/02/ruby-on-rails-application-migrating.html Tomcat]
* [[JRuby on Rails on Jetty|Jetty]]
* [[JRuby on Rails on WebSphere|WebSphere]]
* [http://wiki.oracle.com/page/JRuby+on+Rails%3A+Deploying+to+Oracle+Containers+for+Java+EE+%28OC4J%29 Oracle OC4J]
* [[JRuby on Rails on Oracle Weblogic|Oracle WebLogic]]
* [http://oddthesis.org/theses/jboss-rails JBoss]

==== Glassfish v3 Deployment ====
The Glassfish gem by Vivek Pandey enables you to easily deploy a JRuby on Rails application to the Glassfish v3 modularized Java application server. The Glassfish gem wraps the essential technologies in 3 MB and allows you to run your application using a traditional approach, as if you were running Mongrel, Rack, and so on.

==== Traditional Rails Deployment ====

* [http://puma.io/ PUMA] (works with MRI as well as JRuby)
* [https://github.com/trinidad/trinidad Trinidad] (based on Tomcat)
* [http://torquebox.org TorqueBox] (based on JBoss)
* [http://modrails.com Passenger]
* [http://en.wikipedia.org/wiki/Mongrel_(web_server) Mongrel Web Server entry on Wikipedia]
* [http://swiftiply.swiftcore.org/mongrel.html Swiftiply with Mongrel]
* [http://www.webrick.org/ WEBrick website]
* [http://en.wikipedia.org/wiki/WEBrick WEBrick entry on Wikipedia]
* [http://microjet.ath.cx/WebWiki/WEBrick.html Gnome's Guide to WEBrick]

=== Miscellaneous Rails Topics ===

* [http://blarg.slackworks.com/posts/jndi-with-rails Blog entry on setting up JNDI]
* [[RailsJSR286Portlets|Write Portlets using Rails]]
* [http://blarg.slackworks.com/posts/jruby-service-in-rails JRuby Services in Rails]
