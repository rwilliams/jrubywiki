Performance Tuning
==================
JRuby supports a number of options to help you tune performance. They range from turning on experimental features to turning off inefficient emulations of Ruby features.This document describes these options and their effects.<br/><br/>

Enabling invokedynamic
----------------------

Java 7 brings with it an important new feature called invokedynamic, which greatly improves JRuby's performance on VMs that support it.

However, current released versions of OpenJDK 7 sometimes error out or fail to optimize code as well as they should. In order to provide a consistent JRuby experience, the use of invokedynamic is off by default on Java 7.

The use of invokedynamic is enabled by default when running on OpenJDK 8 builds, since the issues from OpenJDK 7 have been resolved. However, current builds (b55) have not yet had optimization work, putting their performance usually ahead of non-invokedynamic but behind OpenJDK 7 invokedynamic. This will improve over time.

For applications that do not run into the errors or degraded performance, invokedynamic is recommended for maximum performance. It can be forcibly enabled by passing `-Xcompile.invokedynamic=true` to JRuby (or in JRUBY_OPTS) or by setting the jruby.compile.invokedynamic=true property at the JVM level. We recommend testing your application thoroughly with invokedynamic enabled before enabling it in production settings.

Profiling an application
------------------------
JRuby has a built-in profiler that can be used to profile the entire stack all the way down to a single line of code.

* See [[Profiling JRuby]] for more details.

The JVM also has a few profilers built in:

* --sample passed to JRuby or -Xprof passed to the JVM enables the JVM's sampling profiler. This is a very low-cost (and low accuracy) way to find especially egregious hotspots, like stack trace generation mentioned below.

* -Xrunhprof:cpu=times passed to the JVM enables the JVM's global instrumented profiler. This will slow the application tremendously, but measures actual times of all methods running on the JVM. We generally recommend JRuby's profiler (--profile) before using runhprof:cpu=times, or use of the latter only for small, self-contained tests. Results are dumped to java.hprof.txt on exit. Use -Xrunhprof:help for a full set of options.

* -Xrunhprof passed to the JVM will profile all object allocations and record backtraces for those allocations, so you can see if you're creating too many objects. Results are dumped to java.hprof.txt on exit.

In addition to the built-in profilers, there are many third-party tools (some free, some not) that do excellent jobs of profiling. Some also have specialized support for JRuby applications.

For example, [NetBeans IDE](http://netbeans.org) includes a Java profiler (without any specific JRuby support).  See the tutorial [Profiling JRuby with NetBeans
](http://polycrystal.org/2012/10/26/profiling_jruby_with_netbeans.html) for a walk-through on setting up and using the NetBeans profiler.

### Common profiling hotspots

* Stack trace/backtrace generation: Generating backtraces for exceptions, Kernel#caller, etc are more expensive on JRuby due to the way the JVM optimizes.

* Slow or unoptimized methods in JRuby itself: JRuby's not perfect, and once in a while you'll find an especially slow method. Report it to the JRuby team and we'll try to improve it. Remember: JRuby being slower than other implementations is usually a bug.

* Excessive object churn: Ruby is a very GC-intensive language, but with a little care you can reduce object allocation rates substantially. Look at object/allocation profiles to see if you're creating too much junk.

* Systems that don't stabilize: If you're continuing to define constants and methods during the normal runtime of your application, you are likely invalidating the caches that JRuby (and the JVM) uses to optimize code. An ideal system will stop defining new methods and constants at some point and allow the system to stabilize, also known as reaching *steady state*.

Tuning the compiler
-------------------
JRuby's compiler can be enabled in JIT mode or specified to run before execution.

* See [[JRuby Compiler|JRubyCompiler]] for more details.
* See below for [Compiler Runtime Properties](#compiler_rt_props).
* See below for [JIT Runtime Properties](#jit_rt_props).

Don't enable ObjectSpace
------------------------
ObjectSpace has been disabled by default since version 1.1b1. Some users reenable ObjectSpace, using the `-X+O` flag (previously `+O`), without realizing what a massive performance hit this is. If you are running with ObjectSpace enabled, do not expect any sort of performance.

Check the OpenJDK/SunJDK/OracleJDK Code Cache Size
--------------------------------------------------
The "Hotspot" VM that runs all the above-mentioned JVMs has a hard upper limit on how much native (i.e. JIT-compiled) code it will load before disabling JIT completely. If this disabling happens before the meat of your application has been compiled, it will never compile and your performance will suffer.

There are two options to fix this situation: bump up the code cache size; or enable code cache flushing.

The JVM flags of interest are -XX:ReservedCodeCacheSize=### where the default is 48M, and -XX:+ UseCodeCacheFlushing.

The folks at Atlassian have published a good article on [code cache effects and flushing](http://blogs.atlassian.com/2012/05/codecache-is-full-compiler-has-been-disabled/).

Enable coroutine-based Fibers
-----------------------------

_Note: Support for coroutine backed fibers has been removed and may be added when a future JVM version supports coroutines. See [Differences Between MRI And JRuby](https://github.com/jruby/jruby/wiki/DifferencesBetweenMriAndJruby#continuations-and-fibers). The steps below will not have any effect and are for historic reference._

Enables coroutine support for Ruby 1.9 [Fibers](http://www.ruby-doc.org/core-1.9/Fiber.html). This requires JRuby 1.6.5 or later and a JVM with the coroutine [MLVM](http://openjdk.java.net/projects/mlvm/) patch applied. Binary builds (for Linux) are [available](http://ssw.jku.at/General/Staff/LS/coro/) from the patch author. 

    jruby -J-Djruby.fiber.coroutines=true myscript.rb

Java Virtual Machine (JVM) Settings
-----------------------------------
Except for the JRuby convenience parameter `--server`, all JVM runtime parameters use the `-J` option, followed by the specific JVM setting. For example:

* Heap space settings: `jruby -J-X<heap-space-setting>`
* JRuby runtime settings: `jruby -J-D<runtime-setting>`

All the settings described in the following sections are JVM settings.

### Using the Java Server Virtual Machine
JRuby benefits greatly from running under the Java **Server** VM, which trades startup and early run performance for a much higher level of long term optimization. JRuby will use the server VM of whatever Java version you're running if you use the `--server` parameter, which passes the `-server` flag to the underlying JVM. For example:

 jruby --server myscript.rb

Combining use of the server VM with using the compiler and disabling ObjectSpace generally results in the fastest performance.

**Note:** The `--server` parameter is a convenient shorthand for the JVM parameter  `-J-server`.

### Setting Heap Space Parameters for JRuby
* Maximum heap space:

    jruby -J-Xmx512m

* Initial heap space:

    jruby -J-Xms512m

* Heap space for Young/Eden Garbage Collection:

    jruby -J-Xmn128m

**All together now**

First some suggestions:

* Set the minimum `-Xms` and maximum `-Xmx` heap sizes to the same value.
* Set the `-Xmn` value lower than the `-Xmx` value.

    jruby -J-Xmn512m -J-Xms2048m -J-Xmx2048m -J-server

### Setting JRuby Runtime Properties
To see all the Java system runtime properties for JRuby, enter the following command in the Command window or Terminal window:
 jruby --properties

All these properties can be used to alter runtime behavior for performance or compatibility. Specify them by passing `-J-Dproperty=value` on the command line. For example:

    jruby -J-Djruby.thread.pooling=true myscript.rb

<a name="compiler_rt_props"/>
### Compiler Runtime Properties
JRuby 1.3.1 properties. Specify these properties by passing `-J-Dproperty=value` on the command line.

    jruby.compile.mode=JIT|FORCE|OFF
       Set compilation mode. JIT is default; FORCE compiles all, OFF disables.
    jruby.compile.fastest=true|false
       (EXPERIMENTAL) Turn on all experimental compiler optimizations.
    jruby.compile.frameless=true|false
       (EXPERIMENTAL) Turn on frameless compilation where possible.
    jruby.compile.positionless=true|false
       (EXPERIMENTAL) Turn on compilation that avoids updating Ruby position info.
       Default is false
    jruby.compile.threadless=true|false
       (EXPERIMENTAL) Turn on compilation without polling for "unsafe" thread events. 
       Default is false.
    jruby.compile.fastops=true|false
       (EXPERIMENTAL) Turn on fast operators for Fixnum. Default is false.
    jruby.compile.fastcase=true|false
       (EXPERIMENTAL) Turn on fast case/when for all-Fixnum whens. Default is false.
    jruby.compile.chainsize=<line count>
       Set the number of lines at which compiled bodies are "chained". Default is 500.
    jruby.compile.lazyHandles=true|false
       Generate method bindings (handles) for compiled methods lazily. Default is false.
    jruby.compile.peephole=true|false
       Enable or disable peephole optimizations. Default is true (on).

<a name="jit_rt_props"/>
### JIT Runtime Properties
JRuby 1.3.1 properties. Specify these properties by passing `-J-Dproperty=value` on the command line.

    jruby.jit.threshold=<invocation count>
       Set the JIT threshold to the specified method invocation count. Default is 50.
    jruby.jit.max=<method count>
       Set the max count of active methods eligible for JIT-compilation.
       Default is 4096 per runtime. A value of 0 disables JIT, -1 disables max.
    jruby.jit.maxsize=<jitted method size (full .class)>
       Set the maximum full-class byte size allowed for jitted methods. 
       Default is Integer.MAX_VALUE.
    jruby.jit.logging=true|false
       Enable JIT logging (reports successful compilation). Default is false.
    jruby.jit.logging.verbose=true|false
       Enable verbose JIT logging (reports failed compilation). Default is false.
    jruby.jit.logEvery=<method count>
       Log a message every n methods JIT compiled. Default is 0 (off).
    jruby.jit.exclude=<ClsOrMod,ClsOrMod::method_name,-::method_name>
       Exclude methods from JIT by class/module short name, c/m::method_name,
       or -::method_name for anon/singleton classes/modules. Comma-delimited.

### Native Support Runtime Properties
JRuby 1.3.1 properties. Specify these properties by passing `-J-Dproperty=value` on the command line.

    jruby.native.enabled=true|false
       Enable/disable native extensions (like JNA for non-Java APIs). Default is true.
       (This affects all JRuby instances in a given JVM.)
    jruby.native.verbose=true|false
       Enable verbose logging of native extension loading. Default is false.
    jruby.fork.enabled=true|false
       (EXPERIMENTAL, maybe dangerous) Enable fork(2) on platforms that support it.

<a name="thread_pool_rt_props"/>
### Thread Pooling Runtime Properties
JRuby 1.3.1 properties. Specify these properties by passing `-J-Dproperty=value` on the command line.

    jruby.thread.pool.enabled=true|false
       Enable reuse of native backing threads via a thread pool. Default is false.
    jruby.thread.pool.min=<min thread count>
       The minimum number of threads to keep alive in the pool. Default is 0.
    jruby.thread.pool.max=<max thread count>
       The maximum number of threads to allow in the pool. Default is unlimited.
    jruby.thread.pool.ttl=<time to live, in seconds>
       The maximum number of seconds to keep alive an idle thread. Default is 60.

### Miscellaneous Runtime Properties
JRuby 1.3.1 properties. Specify these properties by passing `-J-Dproperty=value` on the command line.

    jruby.compat.version=RUBY1_8|RUBY1_9
       Specify the major Ruby version to be compatible with; Default is RUBY1_8.
    jruby.objectspace.enabled=true|false
       Enable or disable ObjectSpace.each_object. Default is disabled (false).
    jruby.launch.inproc=true|false
       Set in-process launching of e.g. system('ruby ...'). Default is true
    jruby.bytecode.version=1.5|1.6
       Set bytecode version for JRuby to generate. Default is current JVM version.
    jruby.management.enabled=true|false
       Set whether JMX management is enabled. Default is true.
    jruby.debug.fullTrace=true|false
       Set whether full traces are enabled (c-call/c-return). Default is false.

You can see some GC tunable parameters [here](http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-core/27550).