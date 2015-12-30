JRuby+Truffle is designed to be run with a JVM that has the Graal compiler. The easiest way to get this is by downloading the [GraalVM](Downloading GraalVM), but it also possible to use Graal with JDK 9 EA (early access) builds.

https://jdk9.java.net/download/

You will also need to download a snapshot of `graal.jar` from our academic collaborators at JKU Linz.

http://lafo.ssw.uni-linz.ac.at/nexus/content/repositories/snapshots/com/oracle/graal/graal/

To use the EA build of Java to run JRuby+Truffle, set the JAVACMD environment variable. You will also need to pass several other command line options.

```
$ JAVACMD=path/to/jdk9/bin/java bin/jruby -X+T \
  -J-XX:+UnlockExperimentalVMOptions -J-XX:+EnableJVMCI \
  -J-Djvmci.compiler=graal \
  -J-Xbootclasspath/p:graal.jar \
  ...
```

* `-J-XX:+UnlockExperimentalVMOptions -J-XX:+EnableJVMCI` enables JVMCI - the interface that Graal uses
* `-J-Djvmci.compiler=graal` tells JVMCI to use Graal
* `-J-Xbootclasspath/p:graal.jar` puts Graal on the boot classpath, where it needs to be in order to be used by JVMCI

The JDK EA builds lag behind Graal development a bit so the interfaces are not always compatible. This will stabilise over time. I believe at the moment they are unfortunately incompatible.