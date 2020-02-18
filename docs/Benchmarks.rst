Because JRuby runs on the JVM and has various settings for different optimizations, benchmarking it requires a bit more care than benchmarking non-optimizing implementations of Ruby. This document tries to describe the basics of getting good benchmark numbers with JRuby.

How to Benchmark JRuby
======================

Know your Java VM
-----------------
The first thing to know is what version of Java you're running. Usually running `java -version` at the command line will show you. As of this writing, Java 6 is the recommended version of Java (also called Java 1.6), using the Hotspot JVM (underlying Sun's JDK, OpenJDK, Apple's JDK, and the "IcedTea" JDK on RPM-based systems).

On 32-bit systems, Java 6 Hotspot will support running in two different modes: "client" and "server". The "client" mode is intended for desktop and command-line apps that need to start up quickly and may not run long enough to need deeper optimizations. The "server" mode takes a bit longer to start up and warm up, but provides the best long-run performance. On 64-bit systems. Hotspot generally only runs in "server" mode.

JRuby usually tries to force the underlying JVM to "client" mode, to improve startup performance. This also means that by default JRuby is usually not using the optimized mode of the JVM. To enable the "server" mode from a JRuby command line, just pass `--server` to JRuby as a normal argument.

Also, because both JRuby and the "server" VM delay some optimizations until after gathering information at runtime, the first or second result for a benchmark usually does not represent the long-term performance of JRuby. So benchmarks under a few seconds will usually not show JRuby's full potential, nor will benchmarks that only cycle once.

Avoid Heavy IO
--------------
It is also important to avoid heavy IO (reading/writing to files, sockets, or console) unless you're actually benchmarking IO or it's necessary for the code you are benchmarking. IO skews execution performance tremendously and can produce results that vary based on many system-level factors. IO is also somewhat slower in JRuby, so benchmarks of execution performance will be inaccurate when heavy IO is involved.

Benchmarks
----------

Usable Benchmarks
-----------------

* [The Computer Language Benchmarks Game - Ruby 1.8.7 vs. JRuby 1.1.6](http://shootout.alioth.debian.org/u32q/benchmark.php?test=all&lang=jruby&lang2=ruby&box=1)
* [The Computer Language Benchmarks Game - Ruby 1.9 vs. JRuby 1.1.6](http://shootout.alioth.debian.org/u32q/benchmark.php?test=all&lang=jruby&lang2=yarv&box=1)

Unusable Benchmarks
-------------------

[Marc-André Cournoyer's benchmark against tinyrb](http://macournoyer.com/blog/2009/02/12/tinyrb/)

**Reasons:**

* No Java environment listed
* No performance options used (-server, --fast, etc)
* No information about VM warmup

