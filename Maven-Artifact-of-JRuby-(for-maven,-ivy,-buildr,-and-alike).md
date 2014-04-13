adding the jruby-core artifact to your project

    <project>
    ...
      <dependencies>
        <dependency>
          <groupId>org.jruby</groupId>
          <artifactId>jruby</artifactId>
          <version>1.7.12</version>
          <type>pom</type>
        </dependency>
    ...
    </project>

**NOTE:** it is possible to overwrite a dependency used by maven. for example joda-time or snakeyaml but in general older versions might break jruby !!!

        <dependency>
          <groupId>joda-time</groupId>
          <artifactId>joda-time</artifactId>
          <version>2.4</version>
        </dependency>

to verify your new setup have a look at complete dependency tree
   
       mvn dependency:tree -Dverbose

## using snapshot version of jruby ##

there is maven repository [ci.jruby.org/snapshots/maven](http://ci.jruby.org/snapshots/maven/) which offers the latest snapshots of jruby-1.7.x and jruby-9000.dev. for this you need to add the snapshot repo to your pom.xml (or settings.xml):

     <repositories>
       <repository>
         <id>jruby-snapshots</id>
         <url>http://ci.jruby.org/snapshots/maven/</url>
         <releases>
           <enabled>false</enabled>
         </releases>
         <snapshots>
           <enabled>true</enabled>
         </snapshots>
       </repository>
     </repositories>

now you can use like this:

        <dependency>
          <groupId>org.jruby</groupId>
          <artifactId>jruby</artifactId>
          <version>9000.dev-SNAPSHOT</version>
          <type>pom</type>
        </dependency>
