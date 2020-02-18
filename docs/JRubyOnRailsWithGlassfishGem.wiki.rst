[[Home|&raquo; JRuby Project Wiki Home Page]] &nbsp; &nbsp; [[JRubyOnRails#Glassfish_v3_Deployment|&raquo; JRuby on Rails: GlassFish V3 Deployment]]
<h1>JRuby on Rails With GlassFish Gem</h1>
This page discusses how to use the GlassFish Gem with JRuby 1.3.1 or later to get GlassFish working with a JRuby on Rails application.<br/><br/>

__TOC__

==JRuby 1.3.1 and GlassFish Gem Setup==
Install [http://jruby.org/download JRuby] in <tt>/opt/jruby-1.3.1</tt> and symlink <tt>/opt/jruby</tt> to <tt>/opt/jruby-1.3.1</tt> for future convenience.

 # <tt>cd /opt && curl -O <nowiki>http://url.to/jruby-1.3.1.tar.gz</nowiki></tt>
 # <tt>tar xzvf jruby-1.3.1.tar.gz</tt>
 # <tt>ln -s jruby-1.3.1 jruby</tt>

To install the GlassFish gem (by Vivek Pandey):
 $ gem i glassfish

If you install GlassFish as a non-root user, you should for your own convenience add <tt>~/.gem/jruby/1.8/bin</tt> to your PATH. You should also have JRuby in your PATH. 

If you install GlassFish as root, the GlassFish <tt>bin</tt> will be installed by default in the JRuby <tt>bin</tt> directory.

You could put something like this in your <tt>.bashrc</tt> so it's set whenever you log in:
 $ export JRUBY_HOME=/opt/jruby 
 $ export PATH=~/.gem/jruby/1.8/bin:$JRUBY_HOME/bin:$PATH

== Configure Your GlassFish Instance for Your Rails Application ==
 $ cd /path/to/rails-app
 $ jruby -S gfrake config   # Sets up initial config/glassfish.yml

Now edit <tt>config/glassfish.yml</tt> to configure the instance to your liking. For instance:

<pre name="xml">
 environment: production
 http:
    port: 3000
    contextroot: /

 log:
    # Logging level. Log level 0 to 7. 0:OFF, 1:SEVERE, 2:WARNING, 3:INFO (default), 4:FINE, 5:FINER, 6:FINEST, 7:ALL.
    log-level: 2

 jruby-runtime-pool:
    initial: 1
    min: 1
    max: 1

 daemon:
    enable: true
    pid: tmp/pids/glassfish-production.pid
    jvm-options: -server -Xmx2500m -Xms64m -XX:PermSize=256m -XX:MaxPermSize=256m -XX:NewRatio=2 -XX:+DisableExplicitGC -Dhk2.file.directory.changeIntervalTimer=6000
</pre>

===Running the Appserver===
Running your GlassFish application server is very simple:
 $ jruby -S glassfish 

===Stopping the Appserver===
The application is started with the Akuma wrapper, which by default exits when passed a SIGINT (2) (default value of <tt>kill</tt>). For instance:
 $ kill `cat tmp/pids/glassfish-production.pid`  

==Integrating GlassFish Gem with Solaris Management Facility (SMF)==
In Solaris 10 and OpenSolaris, you get an excellent service management facility called SMF that you can easily configure to run your applications on startup (and much more). It can be compared with Linux's <tt>init.rd / rc.d</tt>, but is much more powerful. 

Importing an application into SMF is not so trivial for first time users, so here is a brief description. For more informaiton, see [http://www.sun.com/bigadmin/content/selfheal/smf-quickstart.jsp Solaris Service Management Facility - Quickstart Guide].

===1. Define a Manifest===
First off you need to define a manifest. This is done with a simple XML file. 

Underneath, you will find a manifest with support of two instances or rails applications. Modify these to your liking. I usually prefer to run applications with a non-root user. I haven't included Access Control Lists (ACLs) to minimize the complexity in the example. 

I usually recommend placing web applications such as Rails somewhere under <tt>/var</tt>. In the example I use <tt>/var/apps</tt>. I also use directories like <tt>/var/rails</tt> and so on, but you can put them in your home directory if you prefer.

<pre name="xml">
<?xml version='1.0'?>
<!DOCTYPE service_bundle SYSTEM '/usr/share/lib/xml/dtd/service_bundle.dtd.1'>
<service_bundle type='manifest' name='glassfish-gem'>
  <service name='network/glassfish-gem' type='service' version='0'>
    <dependency name='fs' grouping='require_all' restart_on='none' type='service'>
      <service_fmri value='svc:/system/filesystem/local'/>
    </dependency>
    <dependency name='net' grouping='require_all' restart_on='none' type='service'>
      <service_fmri value='svc:/network/loopback'/>
    </dependency>
    <dependent name='glassfish-gem_multi-user' restart_on='none' grouping='optional_all'>
      <service_fmri value='svc:/milestone/multi-user'/>
    </dependent>
    <exec_method name='start' type='method' exec='/opt/jruby/bin/glassfish' timeout_seconds='60' />
    <exec_method name='stop' type='method' exec=':kill' timeout_seconds='60' />
    <!-- INSTANCES -->        
    <instance name='my-railsapp_production' enabled='false'>
      <method_context working_directory='/var/apps/my-railsapp/development/my-railsapp'>
        <method_credential user='railsuser' group='daemon' />
        <method_environment>
          <envvar name="PATH" value="/opt/jruby/bin:/usr/bin:/bin" />
        </method_environment>
      </method_context>
    </instance>
    <instance name='MYRAILSAPP_ENVIRONMENT' enabled='false'>
      <method_context working_directory='/FULL/PATH/TO/MY/RAILS/APP/ENVIRONMENT'>
        <method_credential user='RUNASTHISUSER' group='RUNASTHISGROUP' />
        <method_environment>
          <envvar name="PATH" value="/PATH/TO/JRUBY/bin:/usr/bin:/bin" />
        </method_environment>
      </method_context>
    </instance>
  </service>
</service_bundle>
</pre>

===2. Import the Manifest Into SMF===
 $ pfexec svccfg validate config/glassfish-gem.smf.xml
 $ pfexec svccfg import config/glassfish-gem.smf.xml

===3. Start Your Appserver===
 $ pfexec svcadm enable glassfish-gem:my-railsapp_production

To verify that your application started, use <tt>svcs</tt>.
 $ pfexec svcs

You should see something like:
 online   Jun_30  svc:/network/glassfish-gem:my-railsapp_production

If it fails to start, it will say <tt>offline</tt> or <tt>maintenance</tt> instead of <tt>online</tt>. In that case, try <tt>svcs</tt> <tt>-xv</tt> and look at the logfile.

===4. Stop Your Appserver===
 $ pfexec svcadm disable glassfish-gem:my-railsapp_production

===5. Restart Your Appserver===
 $ pfexec svcadm restart glassfish-gem:my-railsapp_production

==Apache 2.2 Frontend== 
On OpenSolaris you can install Apache 2.2 with IPS:
 # pkg install SUNWapch22

On Ubuntu/Debian Linux, you would use apt: (''TODO: verify package name on Ubuntu/Debian.'')
  # aptitude install apache2.2

This example's commands also refer to Solaris. You configure Apache 2.2. the same way in other operating systems, but paths to configurations might vary. For instance on Debian/Ubuntu, Apache config files are placed in<br/><tt>/etc/apache2/sites-available/MY-CONFIG-FILE</tt>, and these are enabled by a symlink in<br/><tt>/etc/apache2/sites-enabled</tt> to the config file.

File <tt>/etc/apache2/2.2/sites/myrailsapp.jruby.org</tt>

<pre name="xml">
<VirtualHost *:80>
  ServerName myrailsapp.jruby.org
  DocumentRoot /var/apps/my-railsapp/production/my-railsapp/public

  <Directory /var/apps/my-railsapp/production/my-railsapp/public>
    Options FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
  </Directory>

  <Proxy balancer://myrailsapp>
    BalancerMember http://127.0.0.1:3000
  </Proxy>

  ProxyPass / balancer://myrailsapp/
  CustomLog /var/log/apache2/myrailsapp_production_apache_access_log combined
</VirtualHost>
</pre>
Since I place my virtual hosts in separate files, I need to add <tt>include</tt> to these at the bottom of the main configuration file <tt>/etc/apache2/2.2/httpd.conf</tt>. If you haven't already configured your Apache server to use Virtual Hosts, you might have to add <tt>NameVirtualHost</tt> to your configuration:
 NameVirtualHost *:80
 Include /etc/apache2/2.2/sites/*

Now just enable (or restart) your Apache 2.2 server: 
 $ pfexec svcadm enable apache22 
 $ pfexec svcadm restart apache22

'''TODO''': ''Nginx frontend, Linux init.rd start/stop script. Formatting.''