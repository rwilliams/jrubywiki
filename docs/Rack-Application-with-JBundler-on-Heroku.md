*Note:* unless you use a `Jarfile` you can just use the regular Ruby buildPack to run a JRuby application on Heroku. This article describes how to have both a `Gemfile` and a `Jarfile` in the same project.

## Using the Ruby Buildpack

Most apps will want to use the standard Ruby buildpack. To do so, you must create a `Rakefile` and add an `assets:precompile` task that executes JBundler:

```ruby
task "assets:precompile" do
  require 'jbundler'
  config = JBundler::Config.new
  JBundler::LockDown.new( config ).lock_down
  JBundler::LockDown.new( config ).lock_down("--vendor")
end
```

Add this file to Git along with your `Jarfile` and `Jarfile.lock`, like so:

```
$ git add Rakefile Jarfile Jarfile.lock
$ git commit -m "Added jbundler config"
```

Then deploy to Heroku as normal:

```
$ git push heroku master
...
remote:        Gems in the groups development and test were not installed.
remote:        Bundled gems are installed into ./vendor/bundle.
remote:        Bundle completed (17.52s)
remote:        Cleaning up the bundler cache.
remote: -----> Precompiling assets
remote:        Running: rake assets:precompile
remote:        Picked up JAVA_TOOL_OPTIONS: -Xmx768m -Djava.rmi.server.useCodebaseOnly=true
remote:        Picked up JAVA_TOOL_OPTIONS: -Xmx768m -Djava.rmi.server.useCodebaseOnly=true
remote:        Picked up JAVA_TOOL_OPTIONS: -Xmx768m -Djava.rmi.server.useCodebaseOnly=true
remote:        jbundler provided classpath:
remote:        ----------------
remote:        jbundler runtime classpath:
remote:        ---------------------------
remote:        /app/.m2/repository/io/netty/netty-all/4.0.28.Final/netty-all-4.0.28.Final.jar
remote:        jbundler test classpath:
remote:        ------------------------
remote:        --- empty ---
remote:        Picked up JAVA_TOOL_OPTIONS: -Xmx768m -Djava.rmi.server.useCodebaseOnly=true
remote:        Picked up JAVA_TOOL_OPTIONS: -Xmx768m -Djava.rmi.server.useCodebaseOnly=true
remote:        complete classpath:
remote:        file:/tmp/build_32bb67a007294c97b4d2c688fc600f50/vendor/ruby-2.2.2-jruby-9.0.0.0/lib/ruby/stdlib/jline/jline/2.11/jline-2.11.jar
remote:        file:/tmp/build_32bb67a007294c97b4d2c688fc600f50/vendor/ruby-2.2.2-jruby-9.0.0.0/lib/ruby/stdlib/readline.jar
remote:        file:/tmp/build_32bb67a007294c97b4d2c688fc600f50/vendor/ruby-2.2.2-jruby-9.0.0.0/lib/ruby/stdlib/psych.jar
remote:        file:/tmp/build_32bb67a007294c97b4d2c688fc600f50/vendor/ruby-2.2.2-jruby-9.0.0.0/lib/ruby/stdlib/org/yaml/snakeyaml/1.14/snakeyaml-1.14.jar
remote:        file:/app/.m2/repository/io/netty/netty-all/4.0.28.Final/netty-all-4.0.28.Final.jar
remote:        Asset precompilation completed (228.80s)
...
remote: Verifying deploy..... done.
```

You will see JBundler run during the pre-compilation step.

## Using the Java Buildpack

This describes how to deploy your JRuby app as a Java application on Heroku. the bridge between the java, jruby and heroku is **Ruby-Maven** which will use the **Gemfile**, the **Jarfile** and produce a **pom.xml** which allows to deploy the application on heroku.

the starting point is the **Mavenfile** from [Rack-Application-with-Ruby-Maven](Rack-Application-with-Ruby-Maven) where you need to add three more declarations for heroku.

to use the Jarfile add

     jarfile :classpath => :java

which tells Ruby-Maven to add to the java-classpath and not to the jruby-classloader. to setup heroku add
     
     properties 'tesla.dump.pom' => 'pom.xml', 'tesla.dump.readOnly' => true
   
     plugin( :dependency, '2.3',
             :phase => :package ) do
       execute_goal( :copy,
                     :artifactItems => [ { :groupId => 'com.github.jsimone',
                                           :artifactId => 'webapp-runner',
                                           :version => '7.0.22',
                                           :destFileName => 'webapp-runner.jar' } ] )
     end

the properties just tells Ruby-Maven to dump a **pom.xml**. the plugin declaration installs the webrunner for heroku. with this you also can start your application locally with

     $ rmvn package
     $ java -jar target/dependency/webapp-runner.jar target/*.war

note the **rmvn** which will generate the pom.xml needed for heroku !

the whole **Mavenfile** looks like this

    gemfile

    packaging :war

    pom( 'org.jruby:jruby', '1.7.13' )
    jar( 'org.jruby.rack:jruby-rack', '1.1.14', 
         :exclusions => [ 'org.jruby:jruby-complete' ] )
   
    jruby_plugin!( :gem,
                   :includeLibDirectoryInResources => true,
				   :includeRubygemsInTestResources => false,
                   :includeRubygemsInResources => true )
				   
    plugin( :war, '2.2',
            :warSourceDirectory => '${basedir}/public',
            :webResources => [ { :directory => '${basedir}',
                                 :targetPath => 'WEB-INF',
                                 :includes => [ 'config.ru' ] } ] )

    jarfile :classpath => :java

    properties 'tesla.dump.pom' => 'pom.xml', 'tesla.dump.readOnly' => true

    plugin( :dependency, '2.3',
            :phase => :package ) do
      execute_goal( :copy,
                    :artifactItems => [ { :groupId => 'com.github.jsimone',
                                          :artifactId => 'webapp-runner',
                                          :version => '7.0.22',
                                          :destFileName => 'webapp-runner.jar' } ] )
    end

now you need a **Procfile** for heroku:

     web:    java $JAVA_OPTS -jar target/dependency/webapp-runner.jar --port $PORT target/*.war

IMPORTANT is to create the heroku application with picked buildpack
     
     heroku create --buildpack https://github.com/heroku/heroku-buildpack-java
     
now you can install you application on heroku the usual way. for more info see also <https://github.com/heroku/java-sample#configure-maven-to-download-webapp-runner>.

when you deploy to heroku maven will download the gems and the jars declared in **Gemfile** and **Jarfile**.

for different application layout like rails see again [Rack-Application-with-Ruby-Maven](Rack-Application-with-Ruby-Maven)