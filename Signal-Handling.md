Threading of Signal handlers
----------------------------

JRuby leverages the JVM's own signal-handling APIs, and most JVMs will route those signal traps to a single thread. This differs from MRI in that signals are not handled on the current thread, and any exceptions raised in them will not propagate out any normal user thread.

If you wish for exceptions raised inside Ruby signal handlers to propagate through the main Ruby thread, you will need to have your signal handler use `Thread#raise` against `Thread.main`.

JVM-occupied Signals
--------------------

JRuby supports the same `Signal` API that CRuby does, but there will be differences due to how various JVMs use those signals internally. Signals that are captured by the JVM will generally not be available for JRuby programs.

In some cases there may be ways to reduce JVM signal usage (e.g. the `-Xrs` flag).

Details about how Hotspot uses signals can be found here: http://www.oracle.com/technetwork/java/javase/signals-139944.html#gbzbl

Details for IBM's J9 JVM are here: https://www.ibm.com/support/knowledgecenter/SSYKE2_8.0.0/com.ibm.java.zos.80.doc/user/signals.html

A quick summary of interesting signals:

* Hotspot is known to use `SEGV` and related signals to optimize null checks, among other things.
* Hotspot may use `USR1` and `USR2` on some platforms.
* Hotspot connects `INT` to a JVM-wide hard shutdown. If you wish to capture this signal, you must trap it yourself; unlike CRuby, JRuby will not automatically bind it to an `Interrupt`-raising handler.
* Most JVMs connect `QUIT` to a dump of thread and basic heap information. If you wish to handle it, trap it yourself (but you will lose the built-in dump ability).
* Unusual platform-specific signals may not be mapped into JRuby, due to the fact that we have no per-platform compile phase.

If you need to use a platform-specific or JVM-occupied signal, try the -Xrs flag. If this does not make the signal available, contact the JRuby team via a Github issue or other community medium.