# adding the jruby-core artifact to your project

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

