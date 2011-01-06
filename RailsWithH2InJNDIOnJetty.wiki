'''This page is grossly out of date'''

In order to have an H2 database in a file be accessible from a web server it in necessary to configure the webserver for JNDI.  Here we use warbler to create a war from an existing rails app and deploy it on Jetty.

1. Install h2 adapter and warbler

  # jruby -S gem install activerecord-jdbch2-adapter warbler -v 0.9.5

These instructions don't appear to work with warbler 0.9.9.

2. Create production database and run migrations

I'm not sure what the most elegant way to do this is.  One way is to edit the database.yml file before creating the database to make it easy to create the database and then modify it again before creating the .war file.  In my case, I simply set the name of the development database to the name of the production database.
To do so, edit database.yml as follows:
<pre>
development:
  adapter: jdbch2
  database: c:/db/my_database_name
  username: my_username
  password: my_password

...

production:
  adapter: jdbch2
  jndi: java:comp/env/my_db
  username: my_username
  password: my_password

</pre>

Run

  #  jruby -S rake db:migrate

3. Create and edit warble.rb

Run

  # jruby -S warble config

This will create a config/warble.rb file.
At about line 36 of this file add
  
  # config.gems << "activerecord-jdbch2-adapter"

and be sure that the line.
  
  # config.gem_dependencies = true

is not commented out.

Near the bottom of the file uncomment and edit the line with config.webxml.jndi

  # config.webxml.jndi = "my_db"

Note: I had better luck with this when the value did not include a slash.  "rails/db" for example didn't work.

4.  Create war file.

Run

  #  jruby -S warble war

5.  Configure Jetty

Unzip your Jetty download to JETTY_HOME.  Copy GEM_HOME/jdbc-h2-1.0.63/lib/h2-1.0.63.jar to JETTY_HOME/lib. In order to enable JNDI, the jetty-plus configuration has to be set up.  One way to do this is to uncomment lines 57-71 in JETTY_HOME/etc/jetty-plus.xml.  Another way would be to create another configuration file following JETTY_HOME/etc/jetty.xml and adding the configuration classes in JETTY_HOME/etc/jetty.xml.  If you go the route of uncommenting the lines in jetty-plus.xml, you will need to create a directory JETTY_HOME/webapps-plus and put your war file there.

You will also need to add the jndi data source, for example by adding a file called my_jetty.xml.

<pre>
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://jetty.mortbay.org/configure.dtd">

<Configure id="Server" class="org.mortbay.jetty.Server">

  <New id="MyDb" class="org.mortbay.jetty.plus.naming.Resource">
    <Arg>my_db</Arg>
    <Arg>
      <New class="org.h2.jdbcx.JdbcDataSource">
        <Set name="URL">jdbc:h2:file:c:/db/my_database_name</Set>
	<Set name="User">my_username</Set>
        <Set name="Password">my_password</Set>
      </New>
    </Arg>
  </New>

</Configure>
</pre>

6. Run the Jetty web server

In the scenario described in step 5 this would be.

  #  JETTY_HOME> java -jar start.jar etc/jetty.xml etc/jetty-plus.xml my_jetty.xml

This will start the application at localhost:8080/my_app_name:

Clearly, "my_app_name", "my_username", "my_password", "c:/db/my_database_name" and "my_db" should be replaced throughout with the appropriate values.
