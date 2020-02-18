[[Home|&raquo; JRuby Project Wiki Home Page]] &nbsp; &nbsp; [[JRubyOnRails#Glassfish_v3_Deployment|&raquo; JRuby on Rails: GlassFish Deployment]]
<h1>JRuby on Rails in GlassFish</h1>
[https://glassfish.dev.java.net/downloads/v2.1-b60e.html GlassFish v2.1] is an open source Java EE 5 application server. 

[https://glassfish.dev.java.net GlassFish v3 Preview] offers early access to Java EE 6 technologies and the new Java EE 6 Web Profile.  Developers building traditional enterprise applications have access to the rich features of the full Java EE 6 platform.  Developers building web applications targeting the Web container can use the Java EE 6 Web Profile.  Because GlassFish v3 uses a microkernel architecture based on OSGi, developers can begin with the Java EE 6 Web Profile and use the Update Center to dynamically upgrade to the full Java EE 6 platform.

Java EE is a well tested deployment environment, and combining the benefits of GlassFish with JRuby provides the best deployment environment for Rails applications. Here are some of the benefits of using GlassFish as the deployment environment for JRuby on Rails applications :
* Use the time-tested Java EE deployment platform.
* Avoid complex Mongrel-based deployment.
* Use <a href="{{project warbler page Home}}">Warbler</a> to bundle your Rails application in a Java Web Application Archive (<tt>.war</tt> file) that you can deploy to a GlassFish instance.
* Java EE and Ruby-on-Rails applications can be easily integrated in one container. This enables you to host JRuby-on-Rails applications in organizations that have already made investment in Java EE.
* Rails applications benefit from the out-of-the-box clustering and high-availability support provided by GlassFish.
* Reuse your database connections with GlassFish's database connection pooling mechanism.
* Leverage the extensive set of Java libraries in your JRuby on Rails applications.. 

The following article provides details about why GlassFish is a preferred development and deployment environment for deploying Rails applications: [http://developers.sun.com/appserver/reference/techart/rails_gf/ Rails powered by GlassFish Application Server].<br/><br/>

__TOC__
== GlassFish v2 ==
GlassFish v2 supports the WAR mode of deployment for JRuby on Rails applications. You can create the <tt>.war</tt> file from your existing Rails applications by using <a href="{{project warbler page Home}}">Warbler</a>.

=== GlassFish v2 Setup ===
Refer to the [http://wiki.glassfish.java.net/Wiki.jsp?page=GettingStartedGuide Getting Started Guide].  At the end of the GlassFish v2 setup steps, ensure that you have the following environment variables defined:
 GLASSFISH_ROOT - Points to the directory where GlassFish v2 is installed
 JRUBY_HOME     - Points to the JRuby version in $GLASSFISH_ROOT/jruby/jruby-1_0_3/jruby-1.0.3.
 PATH           - Ensure that $GLASSFISH_ROOT/bin and $JRUBY_HOME/bin are in this environment variable.

=== Creating a Simple Hello Application ===
If you're ready to deploy your own JRuby on Rails application, skip this section and go to the next section, [[#Deploying_JRuby_on_Rails_Applications_in_GlassFish_v2|Deploying JRuby on Rails Applications in GlassFish v2]].

Here are some steps for creating a very simple JRuby on Rails "hello" application. The application uses the JRuby version that you installed in the "Setup" section above.

 cd $GLASSFISH_ROOT/jruby/samples
 jruby -S rails hello
 jruby script/generate controller say message

The last line creates a controller <tt>say</tt> with a view <tt>message</tt>.

=== Deploying JRuby on Rails Applications in GlassFish v2===
This section describes how to deploy the "hello" application to GlassFish v2 in both ''Standalone Mode'' and ''Shared Mode''. 

'''Note:''' In the instructions listed below, change the directory name if you are deploying your own JRuby on Rails application.

If you already have a <tt>.war</tt> file for your JRuby on Rails application (created using <a href="{{project warbler page Home}}">Warbler</a> or [[GoldSpike]]), skip the preliminary instructions and go to "To deploy the Rails application" in either the Standalone Mode section or the Shared Mode section below.

==== Standalone Mode ====
# Go the Rails application directory and enter the following commands:<br/><tt> &nbsp; cd $GLASSFISH_ROOT/jruby/samples/hello<br/> &nbsp; 
$GLASSFISH_ROOT/lib/ant/bin/ant -f $GLASSFISH_ROOT/jruby/install.xml create-standalone</tt> <br/>You can now see a <tt>WEB-INF</tt> directory under the <tt>hello</tt> sample directory that contains <tt>web.xml</tt> and the two directories <tt>lib</tt> and <tt>gems</tt>.
# Before doing a directory deployment of the sample to GlassFish, ensure that the GlassFish application server is running as follows:<br/><tt> &nbsp;  cd ..<br/> &nbsp;  asadmin start-domain</tt><br/>'''Note:''' If the application server is already running, the command safely exits.
# To deploy the Rails application, enter the following commands:<br/><tt> &nbsp;  cd $GLASSFISH_ROOT/jruby/samples<br/>  &nbsp; asadmin deploy hello</tt>
# Access the following URLs in a browser:<br/><tt> &nbsp; <nowiki>http://localhost:8080/hello/say/message</nowiki><br/> &nbsp; <nowiki>http://localhost:8080/hello</nowiki></tt>

==== Shared Mode ====
To deploy an application in shared mode, ensure that you have followed the "Post Installations" steps of the [http://wiki.glassfish.java.net/Wiki.jsp?page=GettingStartedGuide Getting Started Guide]. Please note that the steps have to be done only once for a GlassFish installation.

'''''Note: I don't see a Post Installations section in the GlassFish Getting Started Guide above.'''''
            
You can now see a <tt>WEB-INF</tt> directory under the <tt>hello</tt> sample directory that contains just the file <tt>web.xml</tt>.

'''To deploy the Rails application:'''
 cd $GLASSFISH_ROOT/jruby/samples/hello
 $GLASSFISH_ROOT/lib/ant/bin/ant -f $GLASSFISH_ROOT/jruby/install.xml create-shared

'''Access the following URLs in a browser:''' 
 <nowiki>http://localhost:8080/hello/say/message</nowiki>
 <nowiki>http://localhost:8080/hello</nowiki>

== GlassFish v2 On Microsoft Windows ==
Before deploying Glassfish on Microsoft Windows, be aware that this operating system presents a unique set of challenges.  JRuby doesn't seem to be as well tested on Microsoft Windows as on other platforms. Here are some of the issues you might encounter and some workarounds for them.

=== Windows likes to lock files ===
If you are new to Microsoft Windows, be aware that it likes to "lock" files on you. When you try to delete, move, or change a file, you can get a message saying, "The file is in use." What can cause this message? For example, if you redploy your <tt>.war</tt> file, you often see, "The file <tt>jnidispatch.dll</tt> is in use and wasn't overwritten".  You can ignore this message if you haven't updated your version of JRuby.  But if you have, you should :
* undeploy your app
* stop your domain
* start your domain
* deploy the new version of your app

=== Case of File Names Matters in GlassFish ===
You especially have to watch the case of file names in Microsoft Windows because the operating system isn't case sensitive, so you might forget that GlassFish is. When deploying a <tt>.war</tt> file via script (for example, you layer Ruby over the <tt>asadmin</tt> command set and use rake to deploy the file), watch the case of the file name in your script. 

For example, a file is deployed with the following capitalization:
  asadmin deploy "MyApp.war"

Make sure if you undeploy in your script, you undeploy <tt>MyApp.war</tt> and not <tt>myApp.war</tt>.  

=== Make Sure to Tune Your Glassfish JVM Parameters ===
'''Note:''' This tip applies to all operating systems, not just Microsoft Windows.  

Take the time to profile your application under load and at steady state and find the right set of JVM parameters that will make your app sing.  Glassfish handles a heavy load well, but you have to tune to get the best performance.

<span id="JVM_options"></span>
For example, here's a subset of JVM options in a sample domain xml:
<pre>
 <jvm-options>-XX:SurvivorRatio=2</jvm-options>
 <jvm-options>-XX:MaxPermSize=192m</jvm-options>
 <jvm-options>-server</jvm-options>
 <jvm-options>-Xmx1000m</jvm-options>
 <jvm-options>-Xms1000m</jvm-options>
 <jvm-options>-XX:NewRatio=2</jvm-options>
 <jvm-options>-Djavax.net.ssl.sessionCacheSize=10000</jvm-options>
</pre>

'''Note:''' The <tt>-server</tt> option makes a big difference.

=== Beware the URLs ===
'''Note:''' This tip applies to all operating systems, not just Microsoft Windows.  

GlassFish v2 doesn't like colons (<tt>:</tt>) in URLs.  For example, if you have some REST based web services with resources that have colons in the resource IDs, you'll have to code around them. (FWIW: I believe that V3 and Tomcat don't have this issue).

===Using SSL ===
If you use SSL, front your application server with something like Apache and have it handle the SSL stuff.

Setting up SSL is pretty straightforward.  However, you might encounter some odd issues.  First off, under load you could run out of memory.  When that happened in one installation, diagnosing the problem with Jconsole heap dumps and a Java profiler showed that there was a cache for the SslSessionContextImpl softreferences (which didn't even point to anything), and the cache was growing unbounded. From looking at the source code of the class, it appeared that there was a default timeout of 24 hours to clean up this cache. There didn't seem to be a way to change the timeout, but there was a way to limit the size of the cache. (See the last JVM-OPTION in the [[#JVM_options|list above]], the one that sets the <tt>ssl.sessionCacheSize</tt>).

Setting this option caused another problem to come up for this installation.  It was handling 400 simultaneous users under a supremely heavy load, and the memory was holding great until the application crashed because Microsoft Windows ran out of nonpaged pool memory.  The problem is described at  [http://docs.sun.com/app/docs/doc/820-3530/gfqse?l=fr)&a=view Win 2k3 non paged pool]. Changing the blocking characteristic to <tt>true</tt> for Grizzly stopped the nonpaged pool leak but had the potential to reduce the load the application could handle. The fix was applied only to the SSL listener because the problem didn't occur before switching to SSL.

== GlassFish v3 ==
GlassFish v3 is the next major release of the GlassFish application server, with the focus on modularization, enabling of non-Java EE containers, embedability, Java EE 6, and. 

===JRuby on Rails With GlassFish v3 Gem==
GlassFish v3 has a gem that helps users launch their Ruby on Rails (ROR) applications embedded within the JRuby VM space. For more information on using this gem, see [[JRubyOnRailsWithGlassfishGem|JRuby on Rails With GlassFish Gem]].

===JRuby on Rails With GlassFish v3===
While the GlassFish gem offers a robust and easy-to-use deployment environment, sometimes you need to use Java EE features from inside your Rails application, and for this you need to [http://wiki.glassfish.java.net/Wiki.jsp?page=DeployAndRunRailsOnGlassFishV3 deploy the application to the GlassFish v3 server]. 

== Success Stories ==
* [http://blogs.sun.com/arungupta/entry/jruby_and_glassfish_v2_another WorldxChange Communication NZ]
* [http://blogs.sun.com/arungupta/entry/jruby_on_rails_deployed_on mediacast.sun.com]

== External Links ==
*[http://wiki.glassfish.java.net/Wiki.jsp?page=JRuby GlassFish JRuby Wiki]
*[http://developers.sun.com/appserver/reference/techart/rails_gf/ Rails powered by GlassFish Application Server]
*[https://glassfish.dev.java.net/servlets/SummarizeList?listName=dev Email dev@glassfish.dev.java.net alias] to discuss problems/issues with GlassFish deployments of JRuby on Rails applications.
