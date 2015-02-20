* core/
  * artifacts:
    * org.jruby:jruby-core (maven-central)
    * org.jruby:jruby-core:noasm (maven-central)
    * org.jruby:jruby-core:complete (maven-central, osgi-bundle)
    * lib/jruby.jar (only local)
  * note:
    * core-complete is the same as lib/jruby.jar
    * the osgi bundle part is still pending and actually the pom.xml of jruby-core:complete has still all dependencies which makes it not really useful (needs own pom.xml without any dependencies)
    * noasm moved packages for all asm classes and shaded all dependencies with asm
* truffle/
  * depends: org.jruby:jruby-core
  * artifacts:
    * org.jruby:jruby-truffle (maven-central)
    * lib/jruby-truffle.jar (only local)
* lib/
  * notes:
    * installs default gems
    * no maven-central artifact
* test/
  * notes:
    * can run tests
    * all test profile are designed to run directly from fresh checkout
    * no maven-central artifact
* maven/jruby-stdlib/
  * artifacts:
    * org.jruby:jruby-stdlib (maven-central)
  * note
    * just bundles lib/ into META-INF/jruby.home/lib
    * actuall the right place for this would be lib/ (pending)
* maven/jruby/
  * depends:
    * org.jruby:jruby-core
    * org.jruby:jruby-stdlib
  * artifacts:
    * org.jruby:jruby (maven-central)
  * note: is actually a pom artifact but make usage (pom.xml) more intuitive it comes with an empty jar
* maven/jruby-noasm/
  * depends:
    * org.jruby:jruby-core:noasm
    * org.jruby:jruby-stdlib
  * artifacts:
    * org.jruby:jruby-noasm (maven-central)
  * note: is actually a pom artifact but make usage (pom.xml) more intuitive it comes with an empty jar
* maven/jruby-complete/
  * depends:
    * org.jruby:jruby-core:noasm
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
    * is reuses the jruby-stdlib, i.e. any include patterns for this part is in jruby-stdlib
    * adds a few files from the root directory ./ and their include patterns are in src/main/assembly/jruby.xml

## remark

there is more going on like fixing file permissions or installing bin/jruby, etc