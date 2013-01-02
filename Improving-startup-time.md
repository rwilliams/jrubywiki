JRuby, being built atop an optimizing VM (the JVM), sometimes tends to start up more slowly than the C-based, non-optimizing standard Ruby. This can often be very frustrating when running lots of commands at the command line. This page offers some tips and tricks to improve startup time

Note that some of these tips are not recommended for general execution; for example, disabling JRuby's JIT will obviously reduce runtime performance. Mix and match to suit your needs.

Use the "client" mode of the JVM
================================

The most commonly-used JVM, Hotspot (aka Sun/Oracle's VM, OpenJDK), ships its 32-bit version with two execution modes: "client" and "server". The client mode is designed to start up quickly and not optimize as much. Server mode takes longer to start and warm up code, but eventually optimizes code much more than client mode.

On 64-bit Hotspot, client mode is not available, but you can configure the "tiered" compiler mode to behave similarly.

Enabling client mode (32-bit)
-----------------------------

All 32-bit versions of Hotspot ship with client mode. You can turn it on by passing `-client` to the `java` command, or through JRuby you can pass `--client` or `-J-client`. This is the most effective way to improve startup time.

Client mode on OS X
-------------------

On OS X, where the Hotspot JVM is sometimes (Java 1.6 and lower) shipped in a "universal" 32/64-bit binary, you may also have to pass `-d32` to the `java` command, or `-J-d32` to JRuby, to force the JVM to start up in 32-bit mode. Without this flag, the JVM may start in 64-bit mode, which does not have a "client" mode.

Tiered compilation (64-bit)
---------------------------

When running a 64-bit-only Hotspot JVM, there is no client mode, and the -client flag will not change execution in any way. In these environments, you can get the same effect as client mode by passing two flags: `-XX:+TieredCompilation -XX:TieredStopAtLevel=1` (prefixed with -J if using the `jruby` command, as usual). This forces the "tiered" compiler into a mode that starts up fast and does not optimize beyond what "client" would. These flags have been tested on Java 6 (OpenJDK 1.6) through Java 8 (OpenJDK 1.8) and like "client" mode they appear to provide the best startup times.

If you are unable to use a 32-bit JVM, we recommend enabling the tiered compiler and limiting it to level 1 for best startup.

Regenerate the shared archive
=============================

Starting with Java 5, the HotSpot JVM has included a feature known as Class Data Sharing (CDS). Originally created by Apple for their OS X version of HotSpot, this feature loads all the common JDK classes as a single archive into a shared memory location. Subsequent JVM startups then simply reuse this read-only shared memory rather than reloading the same data again. It's a large reason why startup times on Windows and OS X have been so much better in recent years, and users of those systems may be able to ignore this tip.

On Linux, however, the shared archive is often *never* generated, since installers mostly just unpack the JVM into its final location and never run it. In order to force your system to generate the shared archive, run the following command (as a user with write permissions to Java's install directory): `java -Xshare:dump`

Many Linux systems will install Java but not generate or include the CDS archive. Forcing it to regenerate can improve startup quite a bit.

Disable JRuby's JIT
===================

An interesting side effect of JRuby's JIT is that it sometimes actually slows execution for really short runs. The compilation isn't free, obviously, nor is the cost of loading, verifying, and linking the resulting JVM bytecode. If you have a very short command that touches a lot of code, you might want to try disabling or delaying the JIT.

Disabling is easy: pass the `-X-C` flag to JRuby or set the jruby.compile.mode property to "OFF" by passing `-Djruby.compile.mode=OFF` to the `java` command.

Avoid spawning "sub-rubies"
===========================

It's a fairly common idiom for Rubyists to spawn a Ruby subprocess using `Kernel#system`, `Kernel#exec`, or backquotes. For example, you may want to prepare a clean environment for a test run. That sort of scenario is perfectly understandable, but spawning many sub-rubies can take a tremendous toll on overall runtime.

When JRuby sees a `#system`, `#exec`, or backquote starting with `ruby`, we will attempt to run it in the same JVM using a new JRuby instance *if* you pass the flag -J-Djruby.launch.inproc=true.  Because we have always supported "multi-VM" execution (where multiple isolated Ruby environments share a single process), this can make spawning sub-Rubies considerably faster. This is, in fact, how JRuby's Nailgun support (more on that later) keeps each Jruby JVM "clean" with multiple JRuby command executions. But even though this can improve performance, there's still a cost for starting up those JRuby instances, since they need to have fresh, clean core classes and a clean runtime.

The worst-case scenario is when we detect that we can't spin up a JRuby instance in the same process, such as if you have shell redirection characters in the command line (e.g. `system 'ruby -e blah > /dev/null'`). In those cases, we have no choice but to launch an entirely new JRuby process, complete with a new JVM, and you'll be paying the full zero-to-running cost.  This is also default behavior.

If you're able, try to limit how often you spawn "sub-rubies" or use tools like Nailgun or spec-server to reuse a given process for multiple hits.

Do less at startup
==================

This is a difficult tip to follow, since often it's not your code doing so much at startup (and usually it's RubyGems itself--avoid if possible). One of the sad truths of JRuby is that because we're based on the JVM, and the JVM takes a while to warm up, code executed early in a process runs a lot slower than code executed later. Add to this the fact that JRuby doesn't JIT Ruby code into JVM bytecode until it's been executed a few times, and you can see why cold performance is not one of JRuby's strong areas.

It may seem like delaying the inevitable, but doing less at startup can have surprisingly good results for your application. If you are able to eliminate most of the heavy processing until an application window starts up or a server starts listening, you may avoid (or spread out) the cold performance hit. Smart use of on-disk caches and better boot-time algorithms can help a lot, like saving a cache of mostly-read-only data rather than reloading and reprocessing it on every boot.

Try using Nailgun
=================

In JRuby 1.3, we officially shipped support for Nailgun. Nailgun is a small library and client-side tool that reuses a single JVM for multiple invocations. With Nailgun, small JRuby command-line invocations can be orders of magnitude faster.

To use Nailgun, you need to first start a server instance (probably in the background) by passing `--ng-server` to JRuby. Subsequent commands can use that server instance by passing `--ng` to JRuby.

Nailgun seems like a magic bullet, but unfortunately it does little to help certain common cases like booting RubyGems or starting up Rails (such as when running tests). It also can't help cases where you are causing lots of sub-rubies to be launched, and if you have long-running commands the usual Control-C may not be able to shut them down on the server. Your best bet is to give it a try and let us know if it helps.

Help us find bottlenecks
========================

The biggest advances in startup-time performance have come from users like you investigating the load process to see where all that time is going. If you do a little poking around and find that particular libraries take unreasonably long to start (or just do too much at startup) or if you find that startup time seems to be limited by something other than CPU (like if your hard drive starts thrashing madly or your memory bus is being saturated) there may be improvements possible in JRuby or in the libraries and code you're loading. Dig a little...you may be surprised what you find.

Here's a few JRuby flags that might help you investigate:

* `--profile` and `--profile.graph` turn on JRuby's built-in profiler in "flat" and "graph" modes. Often there will be a slow Ruby-level operation during startup that can be fixed or omitted.
* `--sample` turns on the JVM's sampling profiler. It's not super accurate, but if there's some egregious bottleneck it should rise to the top. This can be useful to find especially slow code in JRuby itself.
* `-J-Xrunhprof:cpu=times` turns on the JVM's instrumented profiler, saving profile results to java.hprof.txt. This slows down execution tremendously, but can give you more accurate low-level timings for JRuby and JDK code.
* `-J-Djruby.debug.loadService.timing=true` turns on timing of all requires, showing fairly accurately where boot-time load costs are heaviest. If there are files that take especially long, there may be a good reason for it.
* On Windows, where you may not have a "time" command, pass `-b` to JRuby (as in `jruby -b ...`) to print out a timing of your command's runtime execution (excluding JVM boot time).

Use a splashscreen
===================

If yours is an end user app with a GUI, you can instruct java to display a splash screen while it loads the JVM.  This can help alleviate the pain (to end users) of the startup time.  The jruby parameter is something like  -J-splash=XXX

When all else fails
===================

If none of these tips gets your startup time to a manageable level, you should connect with JRuby devs and users by posting to the [JRuby Users mailing list](http://xircles.codehaus.org/lists/user@jruby.codehaus.org). They may be able to help with tips specific to your case, or help you investigate if there's a startup-impacting bug that can be fixed.