* core/
  * artifacts:
    * org.jruby:jruby-core (maven-central) (use any profile to build it locally and to get the asm classes shaded and repackage - -Pmain or -Pcomplete)
    * lib/jruby.jar (only local)
* lib/
  * artifacts:
    * org.jruby:jruby-stdlib (maven-central)
  * notes:
    * installs default gems
    * just bundles lib/ into META-INF/jruby.home/lib
* test/
  * notes:
    * can run tests
    * all test profile are designed to run directly from fresh checkout
    * no maven-central artifact
* maven/jruby/
  * depends:
    * org.jruby:jruby-core
    * org.jruby:jruby-stdlib
  * artifacts:
    * org.jruby:jruby (maven-central)
  * note: is actually a pom artifact but make usage (pom.xml) more intuitive it comes with an empty jar
* maven/jruby-complete/
  * depends:
    * org.jruby:jruby-core
    * org.jruby:jruby-stdlib
  * artifacts:
    * org.jruby:jruby-complete (maven-central, osgi-bundle)
  * note: shaded all dependencies and pom.xml has no declared dependencies
* maven/jruby-jars/
  * depends:
    * lib/jruby.jar
    * lib/jruby-truffle.jar
    * org.jruby:jruby-stdlib
  * artifacts:
    * jruby-jars.gem (rubygems)
  * note:
    * the gem will not be deployed via maven
    * the jruby.jar is called jruby-core-complete.jar inside the gem
* maven/jruby-dist/
  * depends:
    * lib/jruby.jar
    * lib/jruby-truffle.jar
    * org.jruby:jruby-stdlib
  * artifacts:
    * org.jruby:jruby-dist:bin (maven-central)
    * org.jruby:jruby-dist:src (maven-central)
  * note:
    * all artifacts come as zip and tar.gz
    * it reuses the jruby-stdlib, i.e. any include patterns for this part is in jruby-stdlib
    * adds a few files from the root directory ./ and their include patterns are in src/main/assembly/jruby.xml
