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

## last . . .

work on a more reliable solution is in progress.