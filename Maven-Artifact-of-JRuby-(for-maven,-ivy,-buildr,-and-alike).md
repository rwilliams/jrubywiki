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

for any other build system please see [search.maven.org](http://search.maven.org/#artifactdetails%7Corg.jruby%7Cjruby%7C1.7.11%7Cjar) for the correct declaration. there is actually also a org.jruby:jruby:jar which is empty, i.e. it is enough to include the pom artifact.

**NOTE:** it is possible to overwrite a dependency used by maven. for example joda-time or snakeyaml but in general older versions might break jruby !!!

        <dependency>
          <groupId>joda-time</groupId>
          <artifactId>joda-time</artifactId>
          <version>2.4</version>
        </dependency>

but on the otherside it can happen that another artifact pulls in an older version of joda-time, snakeyaml, etc and then you need to pick a working version manually as shown above.

to verify your new setup have a look at complete dependency tree
   
       mvn dependency:tree -Dverbose

the verbose option tells you more where and how version conflicts get resolved with maven.

## conflict with ASM version jruby depends on ##

it is possible that the version resolution will pick the wrong version of org.ow2.asm then you can use the artifact org.jruby:jruby-noasm instead.

        <dependency>
          <groupId>org.jruby</groupId>
          <artifactId>jruby-noasm</artifactId>
          <version>1.7.11</version>
          <type>pom</type>
        </dependency>

this one has a few artifacts embedded and relocated the package of org.ow2.asm to org.jruby.org.ow2.asm avoiding any conflict with another asm version.

## using snapshot version of jruby ##

there is maven repository [ci.jruby.org/snapshots/maven](http://ci.jruby.org/snapshots/maven/) which offers the latest snapshots of jruby-1.7.x and jruby-9000.dev. for this you need to add the snapshot repo to your pom.xml (or settings.xml).

TODO ivy and friends

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
