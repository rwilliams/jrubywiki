adding the jruby-core artifact to your project

    <project>
    ...
      <dependencies>
        <dependency>
          <groupId>org.jruby</groupId>
          <artifactId>jruby-core</artifactId>
          <version>1.7.0.RC1</version>
        </dependency>
    ...
    </project>

in case maven picks the wrong version of for example joda-time you always can overwrite that by adding an explicit dependency with the desired version to your pom.xml.

        <dependency>
          <groupId>joda-time</groupId>
          <artifactId>joda-time</artifactId>
          <version>1.6.4</version>
        </dependency>

to verify your new setup have a look at complete dependency tree
   
       mvn dependency:tree -Dverbose

## using gems through maven

there is maven repository which delivers gem artifacts for all gems of rubygems.org: http://rubygems-proxy.torquebox.org/releases

in order to use those gems we need to add the repository to the pom, the gem itself as dependency and we need to tell maven to "install" those gems. the gem maven plugin installs the gems into **target/rubygems** and the plugin tries to keep the outer environment out of the picture as good as possible.

    <project>
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.example</groupId>
      <artifactId>rubygems-in-test-resources</artifactId>
      <version>0.0.0</version>
      <repositories>
        <repository>
          <id>rubygems-release</id>
          <url>http://rubygems-proxy.torquebox.org/releases</url>
        </repository>
      </repositories>
      <dependencies>
        <dependency>
          <groupId>rubygems</groupId>
          <artifactId>haml</artifactId>
          <version>3.0.25</version>
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
          <artifactId>jruby-core</artifactId>
          <version>1.7.0.RC1</version>
        </dependency>
      </dependencies>
      <build>
        <plugins>
           <plugin>
             <groupId>de.saumya.mojo</groupId>
             <artifactId>gem-maven-plugin</artifactId>
             <version>0.29.1</version>
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

