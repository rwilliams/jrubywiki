JRuby+Truffle is designed to be run with a JVM that has the Graal compiler. The easiest way to get this is by downloading the [GraalVM](Downloading GraalVM), but if you are using the `truffle-head` branch this may not be compatible with the version of the Truffle API used, and you may need to build a JVM with Graal for yourself.

https://wiki.openjdk.java.net/display/Graal/Instructions

```
$ git clone https://github.com/graalvm/mx.git
$ export PATH=$PWD/mx:$PATH
$ mkdir graal
$ cd graal
$ git clone https://github.com/graalvm/graal-core.git
$ cd graal-core
$ mx --vm server build
```

To use your build of Graal to run JRuby+Truffle, set the JAVACMD environment variable.

```
$ JAVACMD=graal/jvmci/jdk1.8.0_72/product/bin/java bin/jruby -X+T ...
```

You may need to update the `1.8.0_72` part of that depending on your system's version of Java.

Some more help on building on different distributions of Linux is available at http://mail.openjdk.java.net/pipermail/graal-dev/2015-December/004050.html.