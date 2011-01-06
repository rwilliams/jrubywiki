__TOC__

<h3>This was conceptual work at an early phase of the project.</h3>

<h3>This has now been superseded by [[Rails in a WAR|rails-integration]] which goes a long way towards automating the process.</h3>

-----
Java web applications are typically packaged as WAR files in preparation for distribution and deployment to J2EE servers. It will be useful to be able to package Ruby on Rails applications in a similar form, to enable seamless deployment to Java servers.

The entry point for a web application is through a servlet. This will fill the role of Rails dispatcher.rb, and must be responsible for initializing the JRuby environment, and dispatching the request to an appropriate controller. From this point there are at least two approaches:
# Fill the environment, and call the usual Rails CGI dispatcher
# Implement the Rails request/response interfaces using the Java Servlet API

The first approach has been used successfully by others, however for this I chose to take the second approach, because it allows the existing Java Servlet functionality to be easily leveraged for other uses, such as session handling.

== Quick start ==
The sections below explain how things are setup. To get started quickly, just [http://www.cs.auckland.ac.nz/~robert/general/JRuby_on_Rails_WAR.zip download the package] and follow these steps:
# Modfiy pom.xml and set the artifactId (this is also the name of the WAR)
# Replace src/main/rails with your Rails app
# Modify src/main/webapp/WEB-INF/web.xml, and add a servlet-mapping for each of your routes/controllers
# Build with ''mvn package''
# Try it out with ''mvn jetty:run-war'', and point your browser to http://localhost:8080/

== Project layout ==
I've used a Maven 2 project layout so that Maven can assembly the WAR, however other options are certainly possible.

{| border="1"
! Location
! Description
|-
| [[#Project descriptor|/myrailsapp/pom.xml]]
| the Maven 2 project descriptor
|-
| [[#Rails|/myrailsapp/src/main/rails]]
| this is where your rails application goes
|-
| [[#Web application descriptor|/myrailsapp/src/main/webapp/WEB-INF/web.xml]]
| the webapp descriptor
|}

Additionally [[#Dispatcher|the dispatcher]] code has been placed in these two folders. Ideally this code would be either in a separate library or as part of JRuby, so these paths would not be required.
* [[#Dispatcher|/myrailsapp/src/main/java]]
* [[#Dispatcher|/myrailsapp/src/main/ruby]]

== Project descriptor ==
The project descriptor must assemble the WAR file such that the dispatcher can easily find the Rails code.

We'll require at least the following dependencies:
<pre>
<dependency>
	<groupId>org.jruby</groupId>
	<artifactId>jruby</artifactId>
	<version>0.9.1</version>
</dependency>
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>servlet-api</artifactId>
	<version>2.4</version>
	<scope>provided</scope>
</dependency>
</pre>

We need to direct the war plugin to arrange the extra directories correctly.
<pre>
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-war-plugin</artifactId>
			<version>2.0.2-SNAPSHOT</version>
			<configuration>
				<webResources>
					<!-- Deploy the rails application to WEB-INF/rails, but exclude the public directory -->
					<resource>
						<directory>src/main/rails</directory>
						<targetPath>WEB-INF/rails</targetPath>
						<excludes>
							<exclude>public/</exclude>
						</excludes>
					</resource>
					<!-- Put the public folder in the root, this allows the web server it -->
					<resource>
						<directory>src/main/rails/public</directory>
					</resource>
					<!-- Place the ruby code under WEB-INF/ruby -->
					<resource>
						<directory>src/main/ruby</directory>
						<targetPath>WEB-INF/ruby</targetPath>
					</resource>
				</webResources>
			</configuration>
		</plugin>
		<!-- While not required, this does make testing easy - try "mvn jetty:run-exploded" -->
		<plugin>
			<groupId>org.mortbay.jetty</groupId>
			<artifactId>maven-jetty-plugin</artifactId>
		</plugin>
	</plugins>
</build>
</pre>

Only the 2.0.2+ versions of the war plugin support targetPath, so let's include the snapshot repository.
<pre>
<pluginRepositories>
	<pluginRepository>
		<id>apache.org</id>
		<name>Maven Plugin Snapshots</name>
		<url>http://people.apache.org/maven-snapshot-repository</url>
		<releases>
			<enabled>false</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
	</pluginRepository>
</pluginRepositories>
</pre>

== Rails ==
A standard rails application can be placed under ''/myrailsapp/src/main/rails''.

To do this, just run "rails myrailsapp/src/main/rails".

After this is created, you may need to ensure that any empty directories (e.g. logs), have at least one file in there. One way is to add a README file describing what the directory is for. This is because the war plugin will ignore empty directories.

== Web application descriptor ==
The web application descriptor (web.xml) so be placed in ''/myrailsapp/src/main/webapp/WEB-INF/web.xml''.
<pre>
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
	version="2.4">

	<!-- ruby.home can be set either here, or as a system property, or through the environment variable JRUBY_HOME -->
	<context-param>
		<param-name>jruby.home</param-name>
		<param-value>/Users/regg002/Reference/jruby-0.9.1</param-value>
	</context-param>

	<!-- Declare the Rails dispatcher -->
	<servlet>
		<servlet-name>rails</servlet-name>
		<servlet-class>org.jruby.webapp.RailsServlet</servlet-class>
	</servlet>

	<!-- Any routes we use must be explicitly mapped, this allows the web server to other content as files (e.g. stylesheets) -->
	<servlet-mapping>
		<servlet-name>rails</servlet-name>
		<url-pattern>/rails/*</url-pattern>
	</servlet-mapping>
	<servlet-mapping>
		<servlet-name>rails</servlet-name>
		<url-pattern>/mycontroller/*</url-pattern>
	</servlet-mapping>
	<servlet-mapping>
		<servlet-name>rails</servlet-name>
		<url-pattern>/myothercontroller/*</url-pattern>
	</servlet-mapping>

	<!-- If index.html exists, go there first -->
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
	</welcome-file-list>
</web-app>
</pre>

== Dispatcher ==
This dispatcher is where the real integration work occurs. This is a two-step process:
* RailsServlet, the Java servlet and entry point for the webapp.
* java_servlet_dispatcher.rb, the Ruby implementation of the Rails dispatcher, request and response classes, which makes use of the Servlet API.

=== RailsServlet ===
RailsServlet must perform the following actions:
# Find the Ruby home directory and Rails application
# Setup the initial load paths
# Initialize JRuby
# Initialize Rails by requiring "rails/config/environment" (not "rails/config/environment.rb")
# Create an instance of java_servlet_dispatcher.rb
# On every request, delegate to JavaServletDispatcher.dispatch, passing through HttpServletRequest and HttpServletResponse

=== JavaServletDispatcher ===
JavaServletDispatcher must extend the Rails Dispatcher, however, rather than following the normal process of using the CGI request/response objects, it will construct instances of Rails request and response objects which are implemented using the Servlet API. These wrapper classes are called JavaServletRequest and JavaServletResponse.

JavaSession is also included, this is used by JavaServletRequest, and implements Rails session handling using the HttpSession.
