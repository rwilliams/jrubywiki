JRuby supports the same `Signal` API that CRuby does, but there will be differences due to how various JVMs use those signals internally. Signals that are captured by the JVM will generally not be available for JRuby programs.

In some cases, as with Hotspot (OpenJDK/OracleJDK) there may be ways to reduce JVM signal usage (e.g. the `-Xrs` flag for Hotspot).

* Hotspot is known to use `SEGV` in its garbage collector.
* Hotspot may use `USR1` and `USR2` on some platforms.
* Hotspot connects `INT` to a JVM-wide hard shutdown. If you wish to capture this signal, you must trap it yourself; unlike CRuby, JRuby will not automatically bind it to an `Interrupt`-raising handler.
* Platform-specific signals may not be mapped into JRuby, due to the fact that we have no per-platform compile phase.