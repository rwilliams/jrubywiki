= JRuby on Rails with Spring Support using Maven =

__TOC__

These steps will get you from zero to a working JRuby on Rails app that can access Spring configured beans using maven to manage your dependencies.

== Initial Setup ==
Basic Steps:

# For Rails 2
## Download jruby somewhere
## <code>tar xvfz jruby-bin-1.4.0.tar.gz</code>
## <code>mv jruby-1.4.0/ /Applications # I keep stuff here, but the location doesn't matter</code>
## <code>cd /Applications</code>
## <code>ln -s jruby-1.4.0/ jruby</code>
## Add <code>/Applications/jruby/bin</code> to my path
## <code>jruby -v # should work</code>
## <code>jruby -S gem list</code>
## <code>jruby -S gem install rails</code>
## <code>jruby -S gem install activerecord-jdbcmysql-adapter</code>
## <code>jruby -S gem install jruby-openssl</code>
## <code>jruby -S gem install warbler</code>
## <code>cd ~/workspace</code>
## <code>jruby -S rails app_name</code>
## Commit to version control for easy rollbacks:
### <code>git init .</code>
### <code>git commit -a -m 'initial version' # need a baseline in case we must rollback</code>
## <code>jruby -S script/generate resource yourApp</code>
## ''If you don't want ActiveRecord'': Add <code>config.frameworks -= [ :active_record ]</code> to <code>environment.rb</code>
## create <code>app/views/benchmarks/index.html.erb</code>
## remove <code>public/index.html</code>
## Add <code>map.root :controller =&gt; "benchmarks"</code> to <code>routes.rb</code>
## <code>jruby -S script/server</code>
## open <code>http://localhost:4000</code> - your app is running
# For Rails 3
## Recommend you install [RVM|https://rvm.beginrescueend.com/]
## <code>rvm install jruby</code>
## <code>gem intall rails</code> (Note that RVM handles the "jruby" stuff for you)
## <code>rails new myapp -m http://jruby.org/rails3.rb -d mysql</code>
## <code>cd myapp</code>
## Edit your <code>Gemfile</code> as follows:
##* <code>source 'http://rubygems.org'</code>
##* <code>gem 'rails', '3.0.9'</code>
##* <code>gem 'activerecord-jdbc-adapter'</code>
##* <code>gem 'activerecord-jdbcmysql-adapter'</code>
##* <code>gem 'jdbc-mysql'</code>
##* <code>gem 'jruby-openssl'</code>
##* <code>gem 'jruby-rack'</code>
## <code>bundle install</code>
## Edit your <code>config/database.yml</code>, setting the the <code>adapter:</code> to <code>jdbcmysql</code>
## Edit your <code>config/database.yml</code> setting the appropriate database user/passwords
## <b>Create the databases yourself</b> - there is a bug that prevents <code>rake db:create</code> from working.
## <code>rake db:migrate</code>
## <code>rails server</code>
## Visit <code>localhost:3000</code> and click on "About your Application's Environemnt".  If you don't see any errors, you are good to go
## Commit to version control for easy rollbacks:
### <code>git init .</code>
### <code>git commit -a -m 'initial version' # need a baseline in case we must rollback</code>
## <code>rails generate resource yourApp</code>
## ''If you don't want ActiveRecord'': Add <code>config.frameworks -= [ :active_record ]</code> to <code>environment.rb</code>
## create <code>app/views/benchmarks/index.html.erb</code>
## remove <code>public/index.html</code>
## Add <code>map.root :controller =&gt; "benchmarks"</code> to <code>routes.rb</code>
## <code>rails server</code>
## open <code>http://localhost:3000</code> - your app is running
# Commit to version control

== Basics ==

How to create Java classes in JRuby.

 include Java
 
 blah = java.util.Date.new

== Getting Deps from Maven ==

We don't need Maven a whole lot, if at all, for Rails development, however it is handy to use it to manage our dependencies, especially if you have parent poms or other re-usable components already in your organization.

Courtesy of [http://www.leonardoborges.com/writings/2009/07/01/jruby-on-rails-and-legacy-java-apps-managing-dependencies/ this awesome blog post], we have a pom that will copy all of our needed jar files into the rails app's <code>lib/</code> directory.  Note that you have to run this manually whenever you change your dependencies.  Your <code>pom.xml</code> will look something like:

  <project
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.company</groupId>
  <!-- notice how we specify the packaging to be a war, that way, maven knows 
    where to copy the jar files -->
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <artifactId>railsApp</artifactId>
  <name>railsApp</name>
  <dependencies>
    <dependency>
      <groupId>com.company</groupId>
      <artifactId>java-legacy-app</artifactId>
      <version>1.0-SNAPSHOT</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>railsApp</finalName>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <executions>
          <execution>
            <!-- This tasks only creates a basic structure expected by maven, 
              so it can do its work -->
            <id>create-mock-web-descriptor</id>
            <phase>compile</phase>
            <configuration>
              <executable>/bin/sh</executable>
              <workingDirectory>.</workingDirectory>
              <arguments>
                <argument>-c</argument>
                <argument>
                  mkdir -p src/main/webapp/WEB-INF
                  touch src/main/webapp/WEB-INF/web.xml
             </argument>
              </arguments>
            </configuration>
            <goals>
              <goal>exec</goal>
            </goals>
          </execution>
          <execution>
            <!-- Now in the package phase we copy the jar files that maven 
              put into the fake web app to our rails' lib folder -->
            <id>copy-needed-jars-into-lib</id>
            <phase>package</phase>
            <configuration>
              <executable>/bin/sh</executable>
              <workingDirectory>.</workingDirectory>
              <arguments>
                <argument>-c</argument>
                <argument>
                  rm -f lib/*.jar
                  cp target/railsApp/WEB-INF/lib/*.jar lib
                  rm -rf target/railsApp*
                  rm -rf src
             </argument>
              </arguments>
            </configuration>
            <goals>
              <goal>exec</goal>
            </goals>
          </execution>
          <execution>
            <!-- Here we optionally create the final war file containing 
              our rails app using warbler, doing a small cleanup of the files and folders 
              maven created -->
            <id>create-final-war</id>
            <phase>package</phase>
            <configuration>
              <executable>/bin/sh</executable>
              <workingDirectory>.</workingDirectory>
              <arguments>
                <argument>-c</argument>
                <argument>
                  rm -rf *.war tmp/war
                  jruby -S warble && \
                  mv *.war target/railsApp.war
             </argument>
              </arguments>
            </configuration>
            <goals>
              <goal>exec</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  </project>

Then, to update dependencies:

 mvn package # downloads jars into target/ dir, 
             # then copies them to the lib dir

Finally, Rails needs to load all of these when it starts up.  Create <code>config/initializers/01_java_jars.rb</code>:

 Dir.entries("#{RAILS_ROOT}/lib").sort.each do |entry|
   if entry =~ /.jar$/
     require entry
   end
 end

This basically "loads" all the libs you had maven copy into your app's <code>lib</code> directory.  You'll need to restart
rails to see this.

One thing that wasn't clear to me with JRuby is what "package roots" are available by default.  For example, my company has code with a package rooted at <code>poscore</code>, and ruby code like <code>poscore.model.type.Customer.new</code> just didn't work.  It seems that only certain common top-level packages are automatically available without qualifying, so you can certainly do:

 Customer = Java::poscore.model.Customer
 
 c = Customer.new

I opted to add some methods to <code>application_controller.rb</code>, since I'd be needing these a lot.  There are other methods of simulating Java's <code>import something.*</code> hanging around, but this worked for my purposes:

 # Get the poscore package from Java
 def self.poscore; Java::poscore; end

== Creating a .war ==

One of the coolest things about JRuby and Rails is that you can package up your Rails app as a <code>.war</code> and dump it into a J2EE app server:

 jruby -S warble config # this sets up stuff to make a war
 jruby -S warble        # creates blah.war in local dir

== Setting up Spring ==

Basically, you need to get a list of all the spring configuration files you intend to load.  If your XML configuration files are somewhere in the classpath in a directory called <code>config</code>, you can get a list of them as such

 # Our config files live in src/main/resources/config
 def beans
 ["configurationContext", "otherContext", "dataContext" ].map { |c|
   "classpath:config/#{c}.xml" 
 }.to_java :string
 end

You can then load them as such:

 context = org.springframework.context.support.ClassPathXmlApplicationContext.new(beans)

 # We have a bean named "someService" configured, so we get it
 service = context.getBean("someService")

You should probably put this in an initializer, naming it to ensure it happens after your java jars are loaded.  Using the naming scheme from above, the name <code>config/initializers/02_spring.rb</code> would work as such:

 SPRING_XML_CONFIG_FILES = %w(
     configurationContext 
     migrationContext 
     dataAccessContext 
     dataSourceContext).map { |c|
   "classpath:config/#{c}.xml" 
 }.to_java :string
 
 SPRING_CONTEXT = org.springframework.context.support.
   ClassPathXmlApplicationContext.new(SPRING_XML_CONFIG_FILES)

Again, this assumes that our config files are on the classpath in a <code>config</code> subdirectory.  Your setup may need to be different.

Now, you can use <code>SPRING_CONTEXT</code> anywhere in your code to access beans.  With Ruby's awesome meta-programming, it's not hard to envision some slick <code>method_missing</code> means of getting access to your beans:

 class << context
   alias old_method_missing method_missing
   def method_missing(sym,args)
     if args.empty?
       getBean(sym.toString)
     else
       old_method_missing(sym,args)
     end
   end
 end

 context.someService # gets our someService bean

That's it!  You now have a JRuby on Rails application that can access your existing java-defined Spring beans.