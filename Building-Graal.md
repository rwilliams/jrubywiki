JRuby+Truffle is designed to be run with a JVM that has the Graal compiler. The easiest way to get this is by downloading the [GraalVM](Downloading GraalVM), but for some special cases you may want to build Graal yourself.

Follow the instructions in the Graal core repository.

https://github.com/graalvm/graal-core

The easiest way to run with the version of Graal that you've just build is using the `jt` tool and the `GRAAL_HOME` environment variable.

```
$ GRAAL_HOME=..../graal-core tool/jt.rb run --graal ...normal Ruby arguments here...
```
