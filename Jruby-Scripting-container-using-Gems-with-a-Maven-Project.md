there is maven repository which delivers gem artifacts for all gems of rubygems.org: http://rubygems-proxy.torquebox.org/releases

in order to use those gems we need to add the repository to the pom, the gem itself as dependency and we need to tell maven to "install" those gems. the gem-maven-plugin installs the gems into **target/rubygems** and the plugin tries to keep the external environment out of the picture as good as possible.

    <project>
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.example</groupId>
      <artifactId>rubygems-in-test-resources</artifactId>
      <version>0.0.0</version>
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
               <includeRubygemsInResources>true</includeRubygemsInResources>
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

during the test phase the gem plugin adds the gems to the classloader, i.e. you can use the haml gem inside a jruby scripting container via

     require 'rubygems'
     require 'haml'
     . . . 

when you pack the jar the gems will be added to jar so they are available to the scripting container during runtime. the default for the gem-maven-plugin is not to add those gems so you need the extra configuration to do so - see gem-maven-plugin configuration in the above pom.xml.

when you want to add ruby files to the jar just add them into the directory **src/main/ruby** and the gem-maven-plugin will add those resources to your project.

the default layout is (the target part is default maven working directory but it shows where your gems get installed per default !)

    .
    ├── src
    │ ├── main
    │ │ ├── java
    │ │ └── ruby
    │ └── test
    │     └── java
    └── target
        └── rubygems
            ├── bin
            ├── build_info
            ├── cache
            ├── doc
            ├── gems
            └── specifications

## problems with some classloaders and embedded gems ##

some classloaders (OSGi or some servlet container classloaders or ...) have problems with rubygems itself since it uses globs to parse directories which do not work if the directory is inside a jar file and the classloader does not expose the jar-file location (to parse the jar by jruby).

in such situations the gems needs to packed differently. there is an experimental feature in gem-maven-plugin which works for a good number of gems (not all !!) by including the 'lib' of the gem to jar.

    <project>
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.example</groupId>
      <artifactId>rubygems-in-test-resources</artifactId>
      <version>0.0.0</version>
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
          <scope>provided</scope>
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
             <version>1.0.1-SNAPSHOT</version>
             <configuration>
               <includeGemsInResources>true</includeGemsInResources>
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

note also the gems are using the **provided** scope, since the packed jar does not need those gem dependencies anymore.

another way is to adjust the **LOAD_PATH** before using any gems like in [Pack-a-Rack-Application-with-Ruby-Maven](Pack-a-Rack-Application-with-Ruby-Maven)