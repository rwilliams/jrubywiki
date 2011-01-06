I am using Eclipse and Maven for a number of Java projects. For our development we use "Run Jetty Run" to launch a Jetty servlet really quickly. I'd like to use this combo for JRuby + Rails development - leveraging many of the benefits of these tools that I've worked hard on learning.

0 - Initial Setup

* create a test project
* create a test rails app
* create a pom.xml in your test project

1 - Run Jetty Run

If you don't have it and want to try - you can add the site (and install) to your Eclipse feature sites

 http://run-jetty-run.googlecode.com/svn/trunk/updatesite


2 - Maven

I'm going to assume that you're familiar with Maven. What I've determined you'll need at a minimum is the following additional dependencies:

<pre>
    <dependency>
      <groupId>org.jruby</groupId>
      <artifactId>jruby</artifactId>
      <version>1.1</version>
    </dependency>
    <dependency>
      <groupId>org.jruby.extras</groupId>
      <artifactId>goldspike</artifactId>
      <version>1.6</version>
    </dependency>
</pre>
3 - Setting up your webapp

Create a directory "src/main/webapp/WEB-INF" if necessary

Create a [[Editing Rails Eclipse Jetty Maven web.xml|web.xml ]] web.xml file

4 - Copy (symlink) over your Rails app. I'd prefer to put it into a subdirectory rather than just toss into the webapp dir. So make a subdir like "WEB-INF/rails" and toss your entire rails app in there. You can change the name of this directory in the web.xml file - look for "rails.root"

So you'll have a directory structure like:

 src/main/webapp/WEB-INF/rails/app
 src/main/webapp/WEB-INF/rails/config
 src/main/webapp/WEB-INF/rails/db
 etc.
 plus
 src/main/webapp/WEB-INF/web.xml

5 - Adding your gems

Put them in "src/main/webapp/WEB-INF/gems". Make sure this directory exists before using the commands:

 cd src/main/webapp/WEB-INF
 gem install rails             --install-dir=gems --no-rdoc --no-ri
 gem install activerecord-jdbc-adapter --install-dir=gems --no-rdoc --no-ri
 
 plus add your own

6 - Create your Run Jetty Run configuration

* Open Eclipse's Run Dialog
* Add a new "Jetty Webapp"
** Set the project to this ''Test Project''
** Set the context to something you like like ''jrails''
** Set the WebApp dir to "src/main/webapp"
* Apply
* Run

7 - You should be able to point your browser to http://localhost:8080/jrails/

8 - Watch your console - debug ad naseum - amend these instructions

9 - Use Maven to make your WAR file when ready

 mvn clean war:exploded
or
 mvn clean war:war

''I just figured this out - so I haven't used the config extensively. As I discover more things I will come back and put in debugging tips.''
