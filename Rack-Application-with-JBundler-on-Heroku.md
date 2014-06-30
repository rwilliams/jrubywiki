first note unless you use a **Jarfile** with jbundler you can just use the regular ruby buildPack to run JRuby application on heroku. this here is show an alternative way where you have both a **Gemfile** and a **Jarfile** needs a slighly different approach (or maybe a custom buildPack).

the way to go is to deploy the application as java application on heroku. the bridge between the java, jruby and heroku is **Ruby-Maven** which will use the **Gemfile**, the **Jarfile** and produce a **pom.xml** which allows to deploy the application on heroku.

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