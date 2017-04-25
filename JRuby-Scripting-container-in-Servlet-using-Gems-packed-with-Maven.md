the main ideas from "[Jruby Scripting container using gems with a Maven Project](Jruby-Scripting-container-using-Gems-with-a-Maven-Project)" also apply here. the extra task is to pack the war file which is basically done with using the pom-packaging 'war' which also expects a **web.xml** in **src/main/webapps/WEB-INF**.

    <project>
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.example</groupId>
      <artifactId>rubygems-in-test-resources</artifactId>
      <version>0.0.0</version>
      <packaging>war</packaging>
      <repositories>
        <repository>
          <id>rubygems-releases</id>
          <url>http://rubygems-proxy.torquebox.org/releases</url>
        </repository>
      </repositories>
      <dependencies>
        <dependency>
          <groupId>rubygems</groupId>
          <artifactId>haml</artifactId>
          <version>4.0.5</version>
          <type>gem</type>
        </dependency>
        <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.8.2</version>
          <scope>test</scope>
        </dependency>
        <dependency>
          <groupId>org.jruby</groupId>
          <artifactId>jruby</artifactId>
          <version>1.7.12</version>
          <pom>pom</pom>
        </dependency>
      </dependencies>
      <build>
        <plugins>
           <plugin>
             <groupId>de.saumya.mojo</groupId>
             <artifactId>gem-maven-plugin</artifactId>
             <version>1.0.0</version>
             <configuration>
               <includeRubygemsInResources>true</includeGemsInResources>
             </configuration>
             <executions>
               <execution>
                 <goals>
                   <goal>initialize</goal>
                 </goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </project>

when you pack the war the gems will be added to **WEB-INF/classes** so they are available to the scripting container during runtime.

so the default layout for such a project looks like this:

    .
    ├── src
    │   ├── main
    │   │   ├── java
    │   │   ├── ruby
    │   │   └── webapp
    │   └── test
    │       └── java
    └── target
        └── rubygems
            ├── bin
            ├── build_info
            ├── cache
            ├── doc
            ├── gems
            └── specifications

## servlet container compatibility ##

there are issues with certain servlet classloaders regarding jars which are inside jruby-stdlib.jar especially when it runs the war file as is, i.e. packed and unexploded file.

with some classloaders it could help to unwrap the gems (but that depends on the gems used whether they use only files under their **lib** directory):

		...
           <plugin>
             <groupId>de.saumya.mojo</groupId>
             <artifactId>gem-maven-plugin</artifactId>
             <version>1.0.3</version>
             <configuration>
               <includeGemsInResources>compile</includeGemsInResources>
             </configuration>
             <executions>
               <execution>
                 <goals>
                   <goal>initialize</goal>
                 </goals>
              </execution>
            </executions>
          </plugin>

or use the **$LOAD_PATH** trick from [Rack-Application-with-Ruby-Maven](Rack-Application-with-Ruby-Maven).