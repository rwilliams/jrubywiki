JRuby, being built atop an optimizing VM (the JVM) sometimes tends to start up more slowly than the C-based, non-optimizing standard Ruby. This can often be very frustrating when running lots of commands at the command line. This page offers some tips and tricks to improve startup time

Note that some of these tips are not recommended for general execution; for example, disabling JRuby's JIT will obviously reduce runtime performance. Mix and match to suit your needs.

Use the "client" mode of the JVM
================================

The most commonly-used JVM, Hotspot (aka Sun/Oracle's VM, OpenJDK), ships with two major execution modes: "client" and "server". The client mode is designed to start up quickly and not optimize as much. Server mode takes longer to start and warm up code, but eventually optimizes code much more than client mode.

Enabling client mode
--------------------

All 32-bit versions of Hotspot ship with client mode. You can turn it on by passing -client to the "java" command, or through JRuby you can pass --client or -J-client. This is the most effective way to improve startup time.

Client mode on OS X
-------------------

On OS X, where the Hotspot JVM is shipped in a "universal" 32/64-bit binary, you may also have to pass -d32 to the "java" command, or -J-d32 to JRuby, to force the JVM to start up in 32-bit mode. Without this flag, the JVM may start in 64-bit mode, which does not have a "client" mode.

64-bit non-OS X systems
-----------------------

On non-OS X 64-bit systems, you will not have a universal binary to switch from 64 to 32 bit. If your system is configured to allow 32-bit binaries, a simple way to get access to "client" mode is to install a 32-bit JVM.

Tiered compilation mode
-----------------------

If installing a 32-bit JVM is not an option, there is no way to enable client mode. However, there's another option: "tiered" compilation. Late in the OpenJDK 6 release cycle, a new mode called "tiered compilation" was introduced. Tiered compilation works similar to "client" mode, by starting up quickly and doing earlier, less-optimized compilation of JVM bytecode. Unlike "client", it continues to reoptimize that code, eventually (ideally) reaching "server" mode performance.

Tiered compilation is more effective in the OpenJDK 7 releases, which have now stabilized and have been tested to work well with JRuby. If updating to OpenJDK 7 is an option for you, its tiered compilation mode may help startup considerably.

To enable tiered compilation, you'll want to run in "server" mode, by passing -server to the "java" command or passing --server or -J-server to JRuby, and additionally pass the flag -XX:+TieredCompilation to "java" or the same flag prefixed with -J passed to JRuby.

We are interested in user stories about running in tiered compilation mode, so please let us know how well it works for you.

Regenerate the shared archive
=============================

Starting with Java 5, the HotSpot JVM has included a feature known as Class Data Sharing (CDS). Originally created by Apple for their OS X version of HotSpot, this feature loads all the common JDK classes as a single archive into a shared memory location. Subsequent JVM startups then simply reuse this read-only shared memory rather than reloading the same data again. It's a large reason why startup times on Windows and OS X have been so much better in recent years, and users of those systems may be able to ignore this tip.

On Linux, however, the shared archive is often *never* generated, since installers mostly just unpack the JVM into its final location and never run it. In order to force your system to generate the shared archive, run the following command (as a user with write permissions to Java's install directory): java -Xshare:dump

Many Linux systems will install Java but not generate or include the CDS archive. Forcing it to regenerate can improve startup quite a bit.

Disable JRuby's JIT
===================

An interesting side effect of JRuby's JIT is that it sometimes actually slows execution for really short runs. The compiler isn't free, obviously, nor is the cost of loading, verifying, and linking the resulting JVM bytecode. If you have a very short command that touches a lot of code, you might want to try disabling or delaying the JIT.

Disabling is easy: pass the -X-C flag to JRuby or set the jruby.compile.mode property to "OFF" by passing -Djruby.compile.mode=OFF to the "java" command.

Avoid spawning "sub-rubies"
===========================

It's a fairly common idiom for Rubyists to spawn a Ruby subprocess using Kernel#system, Kernel#exec, or backquotes. For example, you may want to prepare a clean environment for a test run. That sort of scenario is perfectly understandable, but spawning many sub-rubies can take a tremendous toll on overall runtime.

When JRuby sees a #system, #exec, or backquote starting with "ruby", we will attempt to run it in the same JVM using a new JRuby instance. Because we have always supported "multi-VM" execution (where multiple isolated Ruby environments share a single process), this can make spawning sub-Rubies considerably faster. This is, in fact, how JRuby's Nailgun support (more on that later) keeps a single JVM "clean" for multiple JRuby command executions. But even though this can improve performance, there's still a cost for starting up those JRuby instances, since they need to have fresh, clean core classes and a clean runtime.

The worst-case scenario is when we detect that we can't spin up a JRuby instance in the same process, such as if you have shell redirection characters in the command line (e.g. system 'ruby -e blah > /dev/null'). In those cases, we have no choice but to launch an entirely new JRuby process, complete with a new JVM, and you'll be paying the full zero-to-running cost.

If you're able, try to limit how often you spawn "sub-rubies" or use tools like Nailgun or spec-server to reuse a given process for multiple hits.

Do less at startup
==================

This is a difficult tip to follow, since often it's not your code doing so much at startup (and usually it's RubyGems itself). One of the sad truths of JRuby is that because we're based on the JVM, and the JVM takes a while to warm up, code executed early in a process runs a lot slower than code executed later. Add to this the fact that JRuby doesn't JIT Ruby code into JVM bytecode until it's been executed a few times, and you can see why cold performance is not one of JRuby's strong areas.

It may seem like delaying the inevitable, but doing less at startup can have surprisingly good results for your application. If you are able to eliminate most of the heavy processing until an application window starts up or a server starts listening, you may avoid (or spread out) the cold performance hit. Smart use of on-disk caches and better boot-time algorithms can help a lot, like saving a cache of mostly-read-only data rather than reloading and reprocessing it on every boot.

Try using Nailgun
=================

In JRuby 1.3, we officially shipped support for Nailgun. Nailgun is a small library and client-side tool that reuses a single JVM for multiple invocations. With Nailgun, small JRuby command-line invocations can be orders of magnitude faster.

To use Nailgun, you need to first start a server instance (probably in the background) by passing "--ng-server" to JRuby. Subsequent commands can use that server instance by passing "--ng" to JRuby.

Nailgun seems like a magic bullet, but unfortunately it does little to help certain common cases like booting RubyGems or starting up Rails (such as when running tests). It also can't help cases where you are causing lots of sub-rubies to be launched, and if you have long-running commands the usual Control-C may not be able to shut them down on the server. Your best bet is to give it a try and let us know if it helps.

Help us find bottlenecks
========================

The biggest advances in startup-time performance have come from users like you investigating the load process to see where all that time is going. If you do a little poking around and find that particular libraries take unreasonably long to start (or just do too much at startup) or if you find that startup time seems to be limited by something other than CPU (like if your hard drive starts thrashing madly or your memory bus is being saturated) there may be improvements possible in JRuby or in the libraries and code you're loading. Dig a little...you may be surprised what you find.

Here's a few JRuby flags that might help you investigate:

* --profile and --profile.graph turn on JRuby's built-in profiler in "flat" and "graph" modes. Often there will be a slow Ruby-level operation during startup that can be fixed or omitted.
* --sample turns on the JVM's sampling profiler. It's not super accurate, but if there's some egregious bottleneck it should rise to the top. This can be useful to find especially slow code in JRuby itself.
* -J-Xrunhprof:cpu=times turns on the JVM's instrumented profiler, saving profile results to java.hprof.txt. This slows down execution tremendously, but can give you more accurate low-level timings for JRuby and JDK code.
* -J-Djruby.debug.loadService.timing=true turns on timing of all requires, showing fairly accurately where boot-time load costs are heaviest. If there are files that take especially long, there may be a good reason for it.
* On Windows, where you may not have a "time" command, pass -b to JRuby (as in 'jruby -b ...') to print out a timing of your command's runtime execution (excluding JVM boot time).

When all else fails
===================

If none of these tips gets your startup time to a manageable level, you should connect with JRuby devs and users by posting to the [JRuby Users mailing list](http://xircles.codehaus.org/lists/user@jruby.codehaus.org). They may be able to help with tips specific to your case, or help you investigate if there's a startup-impacting bug that can be fixed.