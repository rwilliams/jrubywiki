Configuring JRuby
=================

As of version 1.6.5, JRuby now supports a .jrubyrc file in either the current directory or the user's home directory. The format of this file is a series of key/value pairs, with the key being the property name from ```--properties``` and the value being the value you'd pass on the command line for the same property.

For example, to set the default Ruby version mode to 1.9, you'd use the following line:

```
compat.version=1.9
```

A configuration file that sets 1.9 mode, disables C extensions, and generates full backtraces for the EAGAIN errno would look like:

```
compat.version=1.9
cext.enabled=false
errno.backtrace=true
```

Most properties should be documented in the ```jruby --properties``` output, but for a complete listing look at src/org/jruby/RubyInstanceConfig.java.

A listing from JRuby 1.7 (in dev at time of writing) follows. Some of these are not available on JRuby 1.6.x.

```
These properties can be used to alter runtime behavior for perf or compatibility.
Specify them by passing -X<property>=<value>
  or if passing directly to Java, -Djruby.<property>=<value>
  or put <property>=<value> in .jrubyrc

compiler settings:

   compile.mode=[JIT, FORCE, OFF, OFFIR]
      Set compilation mode. JIT = at runtime; FORCE = before execution. Default is JIT.
   compile.dump=[true, false]
      Dump to console all bytecode generated at runtime. Default is false.
   compile.threadless=[true, false]
      (EXPERIMENTAL) Turn on compilation without polling for "unsafe" thread events. Default is false.
   compile.dynopt=[true, false]
      (EXPERIMENTAL) Use interpreter to help compiler make direct calls. Default is false.
   compile.fastops=[true, false]
      Turn on fast operators for Fixnum and Float. Default is false.
   compile.chainsize=[Integer]
      Set the number of lines at which compiled bodies are "chained". Default is 500.
   compile.lazyHandles=[true, false]
      Generate method bindings (handles) for compiled methods lazily. Default is false.
   compile.peephole=[true, false]
      Enable or disable peephole optimizations. Default is true.
   compile.noguards=[true, false]
      Compile calls without guards, for experimentation. Default is false.
   compile.fastest=[true, false]
      Compile with all "mostly harmless" compiler optimizations. Default is false.
   compile.fastsend=[true, false]
      Compile obj.send(:sym, ...) as obj.sym(...). Default is false.
   compile.inlineDyncalls=[true, false]
      Emit method lookup + invoke inline in bytecode. Default is false.
   compile.fastMasgn=[true, false]
      Return true from multiple assignment instead of a new array. Default is false.
   compile.invokedynamic=[true, false]
      Use invokedynamic on Java 7+. Default is true.

invokedynamic settings:

   invokedynamic.maxfail=[Integer]
      Maximum size of invokedynamic PIC. Default is 2.
   invokedynamic.log.binding=[true, false]
      Log binding of invokedynamic call sites. Default is false.
   invokedynamic.log.constants=[true, false]
      Log invokedynamic-based constant lookups. Default is false.
   invokedynamic.all=[true, false]
      Enable all possible uses of invokedynamic. Default is false.
   invokedynamic.safe=[true, false]
      Enable all safe (but maybe not fast) uses of invokedynamic. Default is false.
   invokedynamic.invocation=[true, false]
      Enable invokedynamic for method invocations. Default is true.
   invokedynamic.invocation.switchpoint=[true, false]
      Use SwitchPoint for class modification guards on invocations. Default is true.
   invokedynamic.invocation.indirect=[true, false]
      Also bind indirect method invokers to invokedynamic. Default is true.
   invokedynamic.invocation.java=[true, false]
      Bind Ruby to Java invocations with invokedynamic. Default is false.
   invokedynamic.invocation.attr=[true, false]
      Bind Ruby attribue invocations directly to invokedynamic. Default is true.
   invokedynamic.invocation.fastops=[true, false]
      Bind Fixnum and Float math using optimized logic. Default is true.
   invokedynamic.cache=[true, false]
      Use invokedynamic to load cached values like literals and constants. Default is true.
   invokedynamic.cache.constants=[true, false]
      Use invokedynamic to load constants. Default is true.
   invokedynamic.cache.literals=[true, false]
      Use invokedynamic to load literals. Default is true.

jit settings:

   jit.threshold=[Integer]
      Set the JIT threshold to the specified method invocation count. Default is 50.
   jit.max=[Integer]
      Set the max count of active methods eligible for JIT-compilation. Default is 4096.
   jit.maxsize=[Integer]
      Set the maximum full-class byte size allowed for jitted methods. Default is 30000.
   jit.logging=[true, false]
      Enable JIT logging (reports successful compilation). Default is false.
   jit.logging.verbose=[true, false]
      Enable verbose JIT logging (reports failed compilation). Default is false.
   jit.dumping=[true, false]
      Enable stdout dumping of JITed bytecode. Default is false.
   jit.logEvery=[Integer]
      Log a message every n methods JIT compiled. Default is 0.
   jit.exclude=[ClsOrMod, ClsOrMod::method_name, -::method_name]
      Exclude methods from JIT. Comma delimited. Default is none.
   jit.cache=[true, false]
      Cache jitted method in-memory bodies across runtimes and loads. Default is true.
   jit.codeCache=[dir]
      Save jitted methods to <dir> as they're compiled, for future runs. Default is null.
   jit.debug=[true, false]
      Log loading of JITed bytecode. Default is false.

intermediate representation settings:

   ir.debug=[true, false]
      Debug generation of JRuby IR. Default is false.
   ir.compiler.debug=[true, false]
      Debug compilation of JRuby IR. Default is false.
   ir.pass.live_variable=[true, false]
      Enable live variable analysis of IR. Default is false.
   ir.pass.dead_code=[true, false]
      Enable dead code elimination in IR. Default is false.
   ir.pass.test_inliner=[String]
      Use specified class for inlining pass in IR. Default is none.

native settings:

   native.enabled=[true, false]
      Enable/disable native code, including POSIX features and C exts. Default is true.
   native.verbose=[true, false]
      Enable verbose logging of native extension loading. Default is false.
   cext.enabled=[true, false]
      Enable or disable C extension support. Default is true.
   ffi.compile.dump=[true, false]
      Dump bytecode-generated FFI stubs to console. Default is false.
   ffi.compile.threshold=[Integer]
      Number of FFI invocations before generating a bytecode stub. Default is 100.
   ffi.compile.invokedynamic=[true, false]
      Use invokedynamic to bind FFI invocations. Default is false.

thread pooling settings:

   thread.pool.enabled=[true, false]
      Enable reuse of native threads via a thread pool. Default is false.
   thread.pool.min=[Integer]
      The minimum number of threads to keep alive in the pool. Default is 0.
   thread.pool.max=[Integer]
      The maximum number of threads to allow in the pool. Default is 2147483647.
   thread.pool.ttl=[Integer]
      The maximum number of seconds to keep alive an idle thread. Default is 60.

miscellaneous settings:

   compat.version=[1.8, 1.9]
      Specify the major Ruby version to be compatible with. Default is 1.8.
   objectspace.enabled=[true, false]
      Enable or disable ObjectSpace.each_object. Default is false.
   launch.inproc=[true, false]
      Set in-process launching of e.g. system('ruby ...'). Default is false.
   bytecode.version=[1.5, 1.6, 1.7]
      Specify the major Ruby version to be compatible with. Default is 1.6.
   management.enabled=[true, false]
      Set whether JMX management is enabled. Default is false.
   jump.backtrace=[true, false]
      Make non-local flow jumps generate backtraces. Default is false.
   process.noUnwrap=[true, false]
      Do not unwrap process streams (issue on some recent JVMs). Default is false.
   reify.classes=[true, false]
      Before instantiation, stand up a real Java class for ever Ruby class. Default is false.
   reify.logErrors=[true, false]
      Log errors during reification (reify.classes=true). Default is false.
   reflected.handles=[true, false]
      Use reflection for binding methods, not generated bytecode. Default is false.
   backtrace.color=[true, false]
      Enable colorized backtraces. Default is false.
   backtrace.style=[normal, raw, full, mri, rubinius]
      Set the style of exception backtraces. Default is normal.
   thread.dump.signal=[USR1, USR2, etc]
      Set the signal used for dumping thread stacks. Default is USR2.
   native.net.protocol=[true, false]
      Use native impls for parts of net/protocol. Default is false.
   fiber.coroutines=[true, false]
      Use JVM coroutines for Fiber. Default is false.

debugging and logging settings:

   debug.loadService=[true, false]
      Log require/load file searches. Default is false.
   debug.loadService.timing=[true, false]
      Log require/load parse+evaluate times. Default is false.
   debug.launch=[true, false]
      Log externally-launched processes. Default is false.
   debug.fullTrace=[true, false]
      Set whether full traces are enabled (c-call/c-return). Default is false.
   debug.scriptResolution=[true, false]
      Print which script is executed by '-S' flag. Default is false.
   errno.backtrace=[true, false]
      Generate backtraces for heavily-used Errno exceptions (EAGAIN). Default is false.
   log.exceptions=[true, false]
      Log every time an exception is constructed. Default is false.
   log.backtraces=[true, false]
      Log every time an exception backtrace is generated. Default is false.
   log.callers=[true, false]
      Log every time a Kernel#caller backtrace is generated. Default is false.
   logger.class=[class name]
      Use specified class for logging. Default is org.jruby.util.log.JavaUtilLoggingLogger.

java integration settings:

   ji.setAccessible=[true, false]
      Try to set inaccessible Java methods to be accessible. Default is true.
   ji.upper.case.package.name.allowed=[true, false]
      Allow Capitalized Java pacakge names. Default is false.
   interfaces.useProxy=[true, false]
      Use java.lang.reflect.Proxy for interface impl. Default is false.
   java.handles=[true, false]
      Use generated handles instead of reflection for calling Java. Default is false.
   ji.newStyleExtension=[true, false]
      Extend Java classes without using a proxy object. Default is false.
   ji.objectProxyCache=[true, false]
      Cache Java object wrappers between calls. Default is true.
```