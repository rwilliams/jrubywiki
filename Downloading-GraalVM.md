JRuby+Truffle is designed to be run with a JVM that has the Graal compiler. The easiest way to get this is via the GraalVM, available from the Oracle Technology Network.

http://www.oracle.com/technetwork/oracle-labs/program-languages/

The GraalVM actually already includes JRuby+Truffle, as a `ruby` command, but this isn't always the latest version so we can also use it to run a version of JRuby+Truffle built from source or installed by any other method used to get JRuby.

To use GraalVM to run JRuby+Truffle, set the `JAVACMD` environment variable.

```
$ JAVACMD=GraalVM-0.9/jre/bin/javao bin/jruby -X+T ...
```

Note that we are using the `javao` command, rather than `java`, due to a bug in GraalVM.

If you are using the `truffle-head` branch, you may need to [build Graal yourself](Building Graal).