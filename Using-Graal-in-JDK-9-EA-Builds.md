JRuby+Truffle is designed to be run with a JVM that has the Graal compiler. The easiest way to get this is by downloading the [GraalVM](Downloading GraalVM), but it also possible to use Graal with JDK 9 EA (early access) builds.

https://jdk9.java.net/download/

You will also need a copy of the Graal and Truffle JARs, and these also need to be built using JDK 9. You can [build this yourself](Building Graal).

To use the EA build of Java to run JRuby+Truffle, set the JAVACMD environment variable. You will also need to pass several other command line options.

```
$ JAVACMD=.../jdk-9-ea+143_osx-x64_bin/Contents/Home/bin/java \
    bin/jruby -X+T \
    -J-XX:+UnlockExperimentalVMOptions \
    -J-XX:+EnableJVMCI \
    -J--add-exports=java.base/jdk.internal.module=com.oracle.graal.graal_core \
    -J--module-path=.../truffle/mxbuild/modules/com.oracle.truffle.truffle_api.jar:.../graal-core/mxbuild/modules/com.oracle.graal.graal_core.jar \
    --no-bootclasspath -e 14
```
