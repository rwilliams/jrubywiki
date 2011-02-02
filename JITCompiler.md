[[Home|&raquo; JRuby Project Wiki Home Page]]  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [[Internals|&raquo; Design: Internals]]
=JIT Compiler Specification=

The JIT compiler in JRuby runs when executing interpreted code.

* The default threshold for a method body to compile is 50 invocations.
* Compilation of a method includes compilation of all contained closures.
* Closure bodies will not compile alone; the containing method must compile.
* JITted methods are single-method transient classes, loaded into their own classloaders using unique class names and thrown away when the method that references the class instance goes away.
