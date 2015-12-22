JRuby+Truffle is designed to be run with a JVM that has the Graal compiler. The easiest way to get this is by downloading the [GraalVM](Downloading GraalVM), but if you are using the `truffle-head` branch this may not be compatible with the version of the Truffle API used, and you may need to build and JVM with Graal for yourself.

https://wiki.openjdk.java.net/display/Graal/Instructions

```
$ hg clone https://bitbucket.org/allr/mx
$ export PATH=$PWD/mx:$PATH
$ mkdir graal
$ cd graal
$ mx sclone http://hg.openjdk.java.net/graal/graal-compiler
$ cd graal-compiler
$ mx --vm server build
```

We can use Graal to run JRuby+Truffle by setting the JAVACMD environment variable.

```
$ JAVACMD=graal/jvmci/jdk1.8.0_66/product/bin/java bin/jruby -X+T ...
```

You may need to update the `1.8.0_66` part of that depending on your system's version of Java.
