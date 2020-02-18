<h1>GoldSpike</h1>
'''Note: GoldSpike is being discontinued. Use <a href="{{project warbler page Home}}">Warbler</a> instead of GoldSpike for packaging JRuby on Rails applications into WAR files.'''

Java web applications are typically packaged as WAR files in preparation for distribution and deployment to Java EE servers. It is useful to be able to package Ruby on Rails applications in a similar form, to enable seamless deployment to Java servers. This is what GoldSpike does.

__TOC__

== Creating a WAR of a Rails application ==

=== Warbler ===
See <a href="{{project warbler page Home}}">Warbler</a> for our currently recommended way to package your Rails applications as a <tt>.war</tt> file for deployment to a Java app server.

See also [[Jruby on Rails on Tomcat]] (Warbler related info).

''Note this section will be expanded in the near future.''

=== GoldSpike Rake Plugin ===
First, install the plugin:
  script/plugin install http://jruby-extras.rubyforge.org/svn/trunk/rails-integration/plugins/goldspike

'''Note:''' The script above doesn't work on JRuby under Windows XP. You'll need to install the [[Running Rails with ActiveRecord-JDBC|ActiveRecord-JDBC]] gem before you can use the rake-tasks below. You install it with the command:
 gem install activerecord-jdbc-adapter --no-rdoc --no-ri

Make sure that the plugin is installed, and run the following command:
 rake war:standalone:create

If you want to run a lot of JRuby web applications on an application server, it might be more efficient to install JRuby on your server's classpath and create a war containing just the application code:
 rake war:shared:create

Try out your application as a web archive using Jetty:
 rake war:standalone:run

== Building GoldSpike from source ==

Check out the source once:
* svn checkout svn://rubyforge.org/var/svn/jruby-extras/trunk/rails-integration
* cd rails-integration
* set GEM_HOME to point to your gem repository, so that the tests can be run (typically something like <jruby home>/lib/ruby/gems/1.8) (use "/" instead of "\" on Windows)
* mvn install

=== Development Environment ===

==== Eclipse ====
The following command will download all the required jar files and create the Eclipse project for you:
 mvn eclipse:eclipse

After this you need to set the M2_REPO variable within Eclipse to point to your local Maven repository. This is typically at <tt>~/.m2/repository</tt>.

==== NetBeans ====
??? 


==== IntelliJ IDEA ====
The following command will download all the required jar files and create the project for you:
 mvn idea:idea 

Alternatively, you can open the pom directly on newer versions of IDEA, which will automatically convert it into a project.

=== How to Debug Problems ===

== Frequently Asked Questions ==

=== Connecting to a database ===
To enable ActiveRecord you'll need to specify a database connection and package appropriate JDBC drivers.

Adding the JDBC driver is done by adding library dependencies into <tt>config/war.rb</tt>. Create it if it doesn't exist.

For example, the MySQL driver can be included in the WAR with the following line:
 maven_library 'mysql', 'mysql-connector-java', '5.0.4'

Next, configure <tt>database.yml</tt> to enable the use of <tt>ActiveRecord-JDBC</tt> as the ActiveRecord database driver. 

'''Note:''' At present WAR files are set to <b>production by default</b>.

Using our MySQL example again, this becomes:
<pre>
production:
  adapter: jdbc
  driver: com.mysql.jdbc.Driver
  url: jdbc:mysql://localhost/sample_production
  username: root
  password:
  host: localhost
</pre>

Update <tt>config/environment.rb</tt>.  See step '9' of [[Running Rails with ActiveRecord-JDBC]].

Now when you use the web application, it should use JDBC drivers from Java to connect to the database.

=== Using a connection from a pool via JNDI ===

Rails-Integration comes with rake tasks to create a war file for the rails app and configure a [http://jetty.mortbay.org jetty]
web server. To configure JNDI support, the following changes are required to be done...

'''1. Add the data source configuration to the <tt>jetty.xml</tt> file.'''

'''Note:''' Every time you run the rake task - war:standalone:run, the <tt>jetty.xml</tt> is recreated. Therefore, I suggest that you modify the rails-integration war plugin library, called <tt>run.rb</tt> and add the entries below in the appropriate place.

For instance, for Oracle data source:

    &lt;new id="MyApp1" class="org.mortbay.jetty.plus.naming.Resource"&gt;
    &lt;arg&gt;jdbc/MyApp&lt;/arg&gt;
    &lt;arg&gt;
    &lt;new class="oracle.jdbc.pool.OracleConnectionPoolDataSource"&gt;
    &lt;set name="URL"&gt;jdbc:oracle:thin:@192.168.2.23:1521:swami&lt;/set&gt;
    &lt;set name="User"&gt;myapp&lt;/set&gt;
    &lt;set name="Password"&gt;myapp&lt;/set&gt;
    &lt;/new&gt;
    &lt;/arg&gt;
    &lt;/new&gt;
 
For the full list of configuration details for various databases, see [http://docs.codehaus.org/display/JETTY/DataSource+Examples Jetty DataSource+Examples].

'''Note:''' As stated in the Jetty documentation, the above entries can be updated in <tt>jetty-env.xml</tt> or <tt>jetty-web.xml</tt> as well. The scope of this configuration depends on where you put this configuration information.

'''2. Now, search for the following section in your <tt>jetty.xml</tt> (or the <tt>run.rb</tt> file):'''
    &lt;New class="org.mortbay.jetty.webapp.WebAppContext"&gt;

Add the following entries after the two &lt;Arg&gt; entries:
    &lt;Set name="ConfigurationClasses"&gt;
    &lt;Array id="plusConfig" type="java.lang.String"&gt;
    &lt;Item&gt;org.mortbay.jetty.webapp.WebInfConfiguration&lt;/Item&gt;
    &lt;Item&gt;org.mortbay.jetty.plus.webapp.EnvConfiguration&lt;/Item&gt;
    &lt;Item&gt;org.mortbay.jetty.plus.webapp.Configuration&lt;/Item&gt;
    &lt;Item&gt;org.mortbay.jetty.webapp.JettyWebXmlConfiguration&lt;/Item&gt;
    &lt;Item&gt;org.mortbay.jetty.webapp.TagLibConfiguration&lt;/Item&gt;
    &lt;/Array&gt;
    &lt;/Set&gt;

'''3. Specify the JNDI Logical Name for the web application.'''

Add the following entries in your <tt>web.xml</tt> under WEB-INF folder of your war file. 

'''Note:''' Every time you run the rake task - war:standalone:run, web.xml is recreated.  Therefore, I suggest that you modify the rails-integration war plugin library <tt>create_war.rb</tt> and add the entries below in the appropriate place.
    &lt;resource-ref&gt;
    &lt;description&gt;My DataSource Reference&lt;/description&gt;
    &lt;res-ref-name&gt;jdbc/MyApp&lt;/res-ref-name&gt;
    &lt;res-type&gt;javax.sql.DataSource&lt;/res-type&gt;
    &lt;res-auth&gt;Container&lt;/res-auth&gt;
    &lt;/resource-ref&gt;

'''Note:''' In the trunk version, the above entries are already added.  All you have to do is setup the JNDI name in the property <tt>datasource_jndi_name</tt> inside <tt>war_config.rb</tt>.

'''4. To add JNDI support, Jetty requires two more libraries: Jetty-Naming and Jetty-Plus.'''

The best way to do this is to modify the rails-integration war plugin library <tt>war_config.rb</tt>. Search for <tt>add_jetty_library</tt> and append the following lines telling the war_config to download the two libraries as well:
    add_jetty_library(maven_library('org.mortbay.jetty', 'jetty-plus', '6.1.1'))
    add_jetty_library(maven_library('org.mortbay.jetty', 'jetty-naming', '6.1.1'))

'''5. Modify your <tt>database.yml</tt> to use JNDI.'''
    production:
      adapter: jdbc
      jndi: java:comp/env/jdbc/MyApp
      driver: oracle

Thats it! Your rails web application is ready to run with JNDI Data source.

=== How do I use a specific JRuby release? ===
Add the following line to <tt>config/war.rb</tt>
 maven_library 'org.jruby', 'jruby-complete', '0.9.9-SNAPSHOT'

=== Which version of Rails is used? ===
If RAILS_GEM_VERSION is set in <tt>environment.rb</tt>, this will be used. Otherwise, the latest installed release of Ruby on Rails is used.

However, you might want to manually specify a version of Rails to use. This can be done by adding a gem dependency to <tt>config/war.rb</tt>, such as:
 add_gem 'rails', '= 1.2.3'

=== Can I use servlet filters? ===
Yes you can. There is an example on how to do this on the [[CAS filter]] page.

=== How do you add JAR files to the resulting webapp? ===
Put them under <tt>lib/java</tt> (create that directory if needed), and they'll propagate to the resulting web application.  

'''Note:''' I had to add a maven_library directive to my <tt>config/war.rb</tt> to make this work. In GoldSpike 1.3, <tt>include_library(name,version)</tt> in <tt>config/war.rb</tt> is supposed to provide the ability to add Java libraries from either <tt>lib/java</tt> or <tt>JRUBY_HOME/lib</tt>, but the functionality has been broken for a while. A patch is available here:<br/>
http://rubyforge.org/tracker/index.php?func=detail&aid=13963&group_id=2014&atid=7859

=== Which Java EE servers can I use ? ===
GoldSpike has been tested on:
* [http://glassfish.dev.java.net GlassFish]
* [http://jetty.mortbay.org/ Jetty]
* [http://tomcat.apache.org/ Tomcat]
* [http://www.bea.com/weblogic/ WebLogic]
* [http://www.ibm.com/developerworks/websphere/techjournal/0801_shillington/0801_shillington.html WebSphere]

=== How do I add dependent gems to the war? ===
Use 'add_gem' in war.rb.  For example:
 add_gem 'rmagick4j', '= 0.3.3'

=== How do I configure the number of requests GoldSpike can handle? ===
The number of simultaneous requests GoldSpike can process is based on the number of JRuby runtimes it creates.

Once GoldSpike has been run, it will create <tt>WEB-INF/web.xml</tt> under your Rails application.
The following <tt>context-params</tt> in <tt>web.xml</tt> can be used to configure GoldSpike:

Maximum number of runtimes:
 jruby.pool.maxActive (defaults to 4)
Minimum number of runtimes:  
 jruby.pool.minIdle (defaults to 2)
Initial number of runtimes:
 jruby.pool.initialSize (defaults to jruby.pool.minIdle)
How often in milliseconds to check if more runtimes are needed:
 jruby.pool.checkInterval (defaults to 1000)

=== What should I do? I get "Could not load Rails. See the logs for more details." error message. ===
This means that an exception was thrown while GoldSpike attempted to initialize the Rails application. This can be caused by a number of problems, for example:
* Failure to connect to the database
* Missing gems from the war
* Missing required jars which are loaded from <tt>environment.rb</tt>

The best way to go about debugging this is to check the container log file. It should show the message and trace for the exception that was encountered.

=== How do I avoid bundling files such as .svn, .DS_Store, etc. into the WAR file? ===
I would LOVE to find the "proper" answer to this. I tried adding the following to <tt>war.rb</tt>, but it only caused an exclusion at the top level, possibly because the file list that finally gets passed to the <tt>jar</tt> command is only a list of the top-level files and directories:
 exclude_files File.join(".","**",".svn")

I ended up having to hack the GoldSpike code to run the following zip command immediately after the WAR file was built (in <tt>packer.rb</tt>), which removes the designated files from the jar manually, throughout the ENTIRE jar file:
 zip -q -d #{os_target_file} \*.svn/\* \*.DS_Store

*os_target_file is the .jar filename
* .DS_Store is the OS X directory metadata file. 
* Backslashes are significant and necessary: See the zip man page!

I can't think of a circumstance when you wouldn't want to exclude these files from a WAR, so I didn't make it an option. Works great.

=== web.xml Notes ===

''After running warbler and looking at the resulting web.xml I was able to infer the following - please add or correct. Thx''

Within your web.xml you can set the following to tailor the WAR structure to your liking. Set inside &lt;web-app&gt; like:

<pre>
 <context-param>
 	<param-name>jruby.standalone</param-name>
 	<param-value>true</param-value>
 </context-param>
</pre>

 jruby.standalone - true/false (what does this do?)
 jruby.session_store - "db"
 jruby.home
 jruby.pool.maxActive - 4
 jruby.pool.minIdle - 2
 jruby.pool.checkInterval - 1000 (ms)
 jruby.pool.maxWait - 30000 (ms)
 
 rails.root - path to rails app within the war file / webapp dir
 rails.env  - development|test|production
 
 files.root - absolute path to public files within the war file / webapp dir
 files.default - servlet id to use if a file cannot be found
 files.prefix - prefix added to static files
 files.welcome -

As with Rails, if a static file exists it will be served, otherwise it will try to dispatch via Rails.

Static file requests are (can be) serviced via <tt>org.jruby.webapp.FileServlet</tt>.

Rails requests are serviced via <tt>org.jruby.webapp.RailsServlet</tt>.

You should map / through the FileServlet
<pre>
 <servlet-mapping>
 	<servlet-name>files</servlet-name>
 	<url-pattern>/</url-pattern>
 </servlet-mapping>
</pre>

Additionally, System Properties you can set:

 jruby.objectspace.enabled
 gem.path
 gem.home

== More Information ==
* [http://blogs.sun.com/arungupta/entry/rails_and_java_ee_integration Rails and Java EE integration - Servlet co-bundled and invoked from Rails app] (Arun Gupta)
* [http://rubyforge.org/mail/?group_id=2014 RubyForge jruby-extras-devel Archives]

== Release roadmap ==

=== 1.5 - January 08 ===
* JRuby 1.1 support
* Rails 2.0 support
* Deprecate plugin, recommend the use of Warbler
* Bug 16108 - HttpOutput flush doesn't send headers - Patch by Matt Burke

=== 1.4 - December 07 ===
* Allow FileServlet to work even if activation.jar is not present at runtime
* Make rails.root param assume it's relative to the webapp
* Make pom download and install rails if necessary
* Added periodical task scheduler

=== 1.3 - August 07 ===
* Update to JRuby 1.0.1
* Support for WebLogic (tested with 9.2)
* Fix PUT, DELETE, etc requests not being passed on to RailsServlet.
* Add support for HEAD requests to FileServlet.
* Fix for a deadlock situation
* Add support for adding arbitrary files to WEB-INF, also looking at timestamps and so on to not add unnecessarily.
* Spring plugin
* Support for caching, by Li Xiao.
* Drop support for preparsing the syntax tree
* Don't add AR-JDBC or Rails gems if they're already added to vendor. By Michael Schubert.
* Servlet configuration templates are now created by a generator. By Bryan Liles.
* Support staggered start up of background tasks
* Make the servlet context directly available to Rails applications as $servlet_context
* Added RailsTaskServlet that makes it possible to run other Rails tasks such as for example ActiveMessaging
* Moved all runtime pool management out into a listener, for proper lifecycle management
* Added support for rails page caching
* Make system environment variables available at request time
* Running an embedded Jetty will now use RAILS_ENV (with a default of 'development') as the rails environment
* Assemble web application in place rather than using tmp/war
* MIT license
* Changed File.install to File.copy to improve performance
* Fix gem install courtesy Jeffrey Damick
* FileServlet can serve directly from an absolute paths

=== 1.2 - May 07 ===
* Bug 9711 - Tomcat 4 support
* Bug 10265 - Http session store
* Tests work with JDK 1.4

=== 1.1.1 - April 23, 2007 ===
* Bug 9321 - Better performance / Glassfish support (no runtime installation)
* Bug 9654 - JDK 1.4 support
* Enhancement 9999 - Support JNDI datasources
* Enhancement 10068 - Edge rails support

* Bug 9320 - NullpointerException in AbstractRailsServlet
* Bug 9394 - Allow deployment to root
* Bug 9419 - Ensure that ARGV is always available
* Enhancement 9710 - add_gem for config file
* Bug 9722 - Allow both symbols and strings for session keys
* Enhancement 10069 - Extra excludes paths
* Bug 10190 - java_library creates corrupt jar-files on windows
* Bug 10265 - fix marshalling issue with http servlet session store
