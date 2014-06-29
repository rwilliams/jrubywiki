## setup

the expected layout for such a project looks like this:

    .
    ├── Gemfile
    ├── config.ru
    ├── lib
    |   └── ....
    └── public

next we need a **Mavenfile** which configures the WAR-plugin and GEM-plugin.

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

now we need the **public/WEB-INF/web.xml**

    <web-app>
      <context-param>
        <param-name>jruby.max.runtimes</param-name>
        <param-value>1</param-value>
      </context-param>
      <context-param>
        <param-name>jruby.min.runtimes</param-name>
        <param-value>1</param-value>
      </context-param>
      <filter>
        <filter-name>RackFilter</filter-name>
        <filter-class>org.jruby.rack.RackFilter</filter-class>
      </filter>
      <filter-mapping>
        <filter-name>RackFilter</filter-name>
        <url-pattern>/*</url-pattern>
      </filter-mapping>
      <listener>
        <listener-class>org.jruby.rack.RackServletContextListener</listener-class>
      </listener>
    </web-app>

for more config options see <https://github.com/jruby/jruby-rack>. now we need to get ruby-maven ;)

the layout changed to

    .
    ├── Gemfile
    ├── Mavenfile
    ├── config.ru
    ├── lib
    |   └── ....
    └── public
        └── WEB-INF
            └── web.xml

almost ready to pack the war file, just make sure Ruby-Maven is installed:

    gem install ruby-maven

maybe bundle the gems first - optional ;-)

    bundle install

after running

    rmvn package

we have (assume you are in 'my_application' directory)

	.
    ....
    └── pkg
        ├── my_application.war
        ├── classes
        └── rubygems
            ├── bin
            ├── build_info
            ├── cache
            ├── doc
            ├── gems
            └── specifications


one note about **bundler**: make sure that the application does **NOT** rely on the auto-require feature of bundler, you do not need the Gemfile inside the war. this example assumes that fact.

## compatiblity with servlet containers ##

rubygems does not work with some classloader. in such a case you can setup the load-path in **WEB-INF/init.rb** like

    $:.unshift "classpath:/gems/dm-transactions-1.2.0/lib"
    $:.unshift "classpath:/gems/fastercsv-1.5.5/lib"
    $:.unshift "classpath:/gems/ixtlan-error-handler-0.4.5/lib"
    $:.unshift "classpath:/gems/dm-validations-1.2.0/lib"

this solves most of the gem related classloader problems !

other classloaders have a problem with those packed jars inside jruby-stdlib. unpacking that jruby-stdlib.jar into **WEB-INF/classes** does help. adding explicit dependencies of the **default gems** to your Gemfile could help. retrieve the list of default gems like this:

    GEM_PATH= GEM_HOME= jruby -S gem list -l

example of how to unpack JRuby-Stdlib:

    gemfile

    packaging :war

	jruby_version = '1.7.13'
    pom( 'org.jruby:jruby', jruby_version
         :exclusions => ['org.jruby:jruby-stdlib' ])
    # needed to install gems for the build itself
    jar( 'org.jruby:jruby-stdlib', jruby_version,
          :scope => :provided )

    jar( 'org.jruby.rack:jruby-rack', '1.1.14', 
         :exclusions => [ 'org.jruby:jruby-complete' ] )
   
    jruby_plugin!( :gem,
                   :includeLibDirectoryInResources => true,
				   :includeRubygemsInTestResources => false,
                   :includeRubygemsInResources => true )
				   
    plugin( :dependency, '2.8', :phase => 'prepare-package'
            :artifactItems => [ { :groupId => 'org.jruby',
                                  :artifactId => 'jruby-stdlib',
                                  :version => jruby_version,
                                  :outputDirectory => '${project.build.outputDirectory}' } ] ) do
      execute_goal( :unpack )
    end

    plugin( :war, '2.2',
            :warSourceDirectory => '${basedir}/public',
            :webResources => [ { :directory => '${basedir}',
                                 :targetPath => 'WEB-INF',
                                 :includes => [ 'config.ru' ] } ] )

that is much **more** then the initial setup.

# heroku

with heroku you install java applications. one advantage of this approach is that you can also take advantage of the jar dependencies from your Jarfile and add two more declarations for heroku. to use the Jarfile add

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

the properties just tells Ruby-Maven to dump a **pom.xml**. the second statement is the webrunner for heroku. with this you also can start your application locally with

     $ rmvn package
     $ java -jar target/dependency/webapp-runner.jar target/*.war

note the **rmvn** which will generate the pom.xml needed for heroku !
now you need a **Procfile** for heroku:

     web:    java $JAVA_OPTS -jar target/dependency/webapp-runner.jar --port $PORT target/*.war

IMPORTANT is to create the heroku application with picked buildpack
     
     heroku create --buildpack https://github.com/heroku/heroku-buildpack-java
     
now you can install you application on heroku the usual way. for more info see also <https://github.com/heroku/java-sample#configure-maven-to-download-webapp-runner>.

when you deploy to heroku maven will download the gems and the jars declared in **Gemfile** and **Jarfile**.

# customize when you have a different application layout

use **app** instead of a **lib** directory add

     resource :directory => '.', :includes => [ 'app/**' ]

and the ```:includeLibDirectoryInResources => true``` is not needed anymore.

for rails you need to add further directories (untested from my side and might need some more attention)

     resource :directory => '.', :includes => [ 'app/**', 'config/**', 'lib/**' ]
