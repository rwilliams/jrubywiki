JRuby Options and .jrubyrc
==========================

JRuby uses a number of JVM properties to help users configure the way it should operate. Most of these properties can be written in a shorthand command-line form using `-Xproperty.name=value`.

You can also put these shorthand properties into a `.jrubyrc` file located either in your home directory or in the current directory (with current directory overriding the version from your home directory).

For example, to log all Ruby exceptions occurring within the runtime, you'd use the following line:

```
log.exceptions=true
```

A configuration file that logs all internal back-trace generation and generates full backtraces for the EAGAIN errno would look like:

```
log.exceptions=true
log.backtraces=true
log.callers=true
errno.backtrace=true
```

Most properties should be documented in the ```jruby --properties``` output, but for a complete listing look at [org/jruby/util/cli/Options.java](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/util/cli/Options.java). All of these options can be passed at the command line using the above `-Xproperty.name=value` format.

A listing from JRuby 9.2.10.0 follows. Some of these are not available in previous JRuby versions.

```
# These properties can be used to alter runtime behavior for perf or compatibility.
# Specify them by passing -X<property>=<value>
#   or if passing directly to Java, -Djruby.<property>=<value>
#   or put <property>=<value> in .jrubyrc
#
# This dump is a valid .jrubyrc file of current settings. Uncomment and modify
# settings to customize.

################################################################################
# parser
################################################################################

# Warn about ambiguous arguments.
# Options: [true, false], Default: true.

#parser.warn.ambiguous_argument=true

# Warn about splat operators being interpreted as argument prefixes.
# Options: [true, false], Default: true.

#parser.warn.argument_prefix=true

# Warn about ignored regex flags being ignored.
# Options: [true, false], Default: true.

#parser.warn.flags_ignored=true

# Warn about interpreting (...) as a grouped expression.
# Options: [true, false], Default: true.

#parser.warn.grouped_expressions=true

# Warn about statements that can never be reached.
# Options: [true, false], Default: true.

#parser.warn.not_reached=true

# Warn about regex literals in conditions.
# Options: [true, false], Default: true.

#parser.warn.regex_condition=true

# Warn about shadowing local variables.
# Options: [true, false], Default: true.

#parser.warn.shadowing_local=true

# Warn about potentially useless expressions in void contents.
# Options: [true, false], Default: true.

#parser.warn.useless_use_of=true


################################################################################
# compiler
################################################################################

# Set the number of lines at which compiled bodies are "chained".
# Default: 500.

#compile.chainsize=500

# Dump to console all bytecode generated at runtime.
# Options: [true, false], Default: false.

#compile.dump=false

# Return true from multiple assignment instead of a new array.
# Options: [true, false], Default: false.

#compile.fastMasgn=false

# Compile with all "mostly harmless" compiler optimizations.
# Options: [true, false], Default: false.

#compile.fastest=false

# Turn on fast operators for Fixnum and Float.
# Options: [true, false], Default: true.

#compile.fastops=true

# Compile obj.__send__(<literal>, ...) as obj.<literal>(...).
# Options: [true, false], Default: false.

#compile.fastsend=false

# Use invokedynamic for optimizing Ruby code.
# Options: [true, false], Default: false.

#compile.invokedynamic=false

# Set compilation mode. JIT = at runtime; FORCE = before execution.
# Options: [JIT, FORCE, OFF], Default: JIT.

#compile.mode=JIT

# Compile calls without guards, for experimentation.
# Options: [true, false], Default: false.

#compile.noguards=false

# Outline when bodies when number of cases exceeds this value.
# Default: 50.

#compile.outline.casecount=50

# Enable or disable peephole optimizations.
# Options: [true, false], Default: true.

#compile.peephole=true

# (EXPERIMENTAL) Turn on compilation without polling for "unsafe" thread events.
# Options: [true, false], Default: false.

#compile.threadless=false


################################################################################
# invokedynamic
################################################################################

# Enable all possible uses of invokedynamic.
# Options: [true, false], Default: false.

#invokedynamic.all=false

# Use invokedynamic to load cached values like literals and constants.
# Options: [true, false], Default: true.

#invokedynamic.cache=true

# Use invokedynamic to load constants.
# Options: [true, false], Default: true.

#invokedynamic.cache.constants=true

# Use invokedynamic to get/set instance variables.
# Options: [true, false], Default: true.

#invokedynamic.cache.ivars=true

# Use invokedynamic to load literals.
# Options: [true, false], Default: true.

#invokedynamic.cache.literals=true

# Use ClassValue to store class-specific data.
# Options: [true, false], Default: false.

#invokedynamic.class.values=false

# Maximum global cache failures after which to use slow path.
# Default: 0.

#invokedynamic.global.maxfail=0

# Use MethodHandles rather than generated code to bind Ruby methods.
# Options: [true, false], Default: false.

#invokedynamic.handles=false

# Enable invokedynamic for method invocations.
# Options: [true, false], Default: true.

#invokedynamic.invocation=true

# Bind Ruby attribute invocations directly to invokedynamic.
# Options: [true, false], Default: true.

#invokedynamic.invocation.attr=true

# Bind Fixnum and Float math using optimized logic.
# Options: [true, false], Default: true.

#invokedynamic.invocation.fastops=true

# Bind Ruby FFI invocations directly to invokedynamic.
# Options: [true, false], Default: true.

#invokedynamic.invocation.ffi=true

# Also bind indirect method invokers to invokedynamic.
# Options: [true, false], Default: true.

#invokedynamic.invocation.indirect=true

# Bind Ruby to Java invocations with invokedynamic.
# Options: [true, false], Default: true.

#invokedynamic.invocation.java=true

# Log binding of invokedynamic call sites.
# Options: [true, false], Default: false.

#invokedynamic.log.binding=false

# Log invokedynamic-based constant lookups.
# Options: [true, false], Default: false.

#invokedynamic.log.constants=false

# Log invokedynamic-based global lookups.
# Options: [true, false], Default: false.

#invokedynamic.log.globals=false

# Maximum call site failures after which to inline cache.
# Default: 1000.

#invokedynamic.maxfail=1000

# Maximum polymorphism of PIC binding.
# Default: 6.

#invokedynamic.maxpoly=6

# Enable all safe (but maybe not fast) uses of invokedynamic.
# Options: [true, false], Default: false.

#invokedynamic.safe=false

# Bind yields directly using invokedynamic.
# Options: [true, false], Default: false.

#invokedynamic.yield=false


################################################################################
# jit
################################################################################

# Run the JIT compiler in a background thread. Off if jit.threshold=0.
# Options: [true, false], Default: true.

#jit.background=true

# (DEPRECATED) Cache jitted method in-memory bodies across runtimes and loads.
# Options: [true, false], Default: true.

#jit.cache=true

# Save jitted methods to <dir> as they're compiled, for future runs.
# Options: [dir].

#jit.codeCache=

# Log loading of JITed bytecode.
# Options: [true, false], Default: false.

#jit.debug=false

# Enable stdout dumping of JITed bytecode.
# Options: [true, false], Default: false.

#jit.dumping=false

# Exclude methods from JIT. <ModClsName or '-'>::<method_name>, comma-delimited.
# Default: .

#jit.exclude=

# Run the JIT compiler while the pure-Ruby kernel is booting.
# Options: [true, false], Default: false.

#jit.kernel=false

# Set JIT class loader to use. UNIQUE class loader per class; SHARED loader for all classes
# Options: [UNIQUE, SHARED, SHARED_SOURCE], Default: UNIQUE.

#jit.loader.mode=UNIQUE

# Log a message every n methods JIT compiled.
# Default: 0.

#jit.logEvery=0

# Enable JIT logging (reports successful compilation).
# Options: [true, false], Default: false.

#jit.logging=false

# Enable verbose JIT logging (reports failed compilation).
# Options: [true, false], Default: false.

#jit.logging.verbose=false

# Set the max count of active methods eligible for JIT-compilation.
# Default: 10000.

#jit.max=10000

# Set the max size (in IR instructions) for a method to be eligible to JIT.
# Default: 1000.

#jit.maxsize=1000

# Set the JIT threshold to the specified method invocation count.
# Default: 50.

#jit.threshold=50


################################################################################
# intermediate representation
################################################################################

# Debug compilation of JRuby IR.
# Options: [true, false], Default: false.

#ir.compiler.debug=false

# Debug generation of JRuby IR.
# Options: [true, false], Default: false.

#ir.debug=false

# Specify file:line of scope to jump to IGV

#ir.debug.igv=

# Specify comma delimeted list of passes to run after inlining a method.

#ir.inline_passes=

# Enable the inliner.
# Options: [true, false], Default: false.

#ir.inliner=false

# Enable the inliner.
# Default: 20.

#ir.inliner.threshold=20

# Report inlining activity.
# Options: [true, false], Default: false.

#ir.inliner.verbose=false

# Specify comma delimeted list of passes to run before JIT.

#ir.jit.passes=

# Specify comma delimeted list of passes to run.

#ir.passes=

# Print the final IR to be run before starting to execute each body of code.
# Options: [true, false], Default: false.

#ir.print=false

# Enable ir.print and include IR executed during JRuby's boot phase.
# Options: [true, false], Default: false.

#ir.print.all=false

# Print the final IR with color highlighting.
# Options: [true, false], Default: false.

#ir.print.color=false

# [EXPT]: Profile IR code during interpretation.
# Options: [true, false], Default: false.

#ir.profile=false

# Read JRuby IR file.
# Options: [true, false], Default: false.

#ir.reading=false

# Debug reading JRuby IR file.
# Options: [true, false], Default: false.

#ir.reading.debug=false

# Implement unboxing opts.
# Options: [true, false], Default: false.

#ir.unboxing=false

# Visualization of JRuby IR.
# Options: [true, false], Default: false.

#ir.visualizer=false

# Write JRuby IR file.
# Options: [true, false], Default: false.

#ir.writing=false

# Debug writing JRuby IR file.
# Options: [true, false], Default: false.

#ir.writing.debug=false


################################################################################
# native
################################################################################

# Dump bytecode-generated FFI stubs to console.
# Options: [true, false], Default: false.

#ffi.compile.dump=false

# Use invokedynamic to bind FFI invocations.
# Options: [true, false], Default: false.

#ffi.compile.invokedynamic=false

# Reify FFI compiled classes.
# Options: [true, false], Default: false.

#ffi.compile.reify=false

# Number of FFI invocations before generating a bytecode stub.
# Default: 100.

#ffi.compile.threshold=100

# Enable/disable native code, including POSIX features and C exts.
# Options: [true, false], Default: true.

#native.enabled=true

# Use native calls to posix_spawn for subprocess execution.
# Options: [true, false], Default: true.

#native.popen=true

# Use pthread_kill to interrupt blocking kernel calls.
# Options: [true, false], Default: true.

#native.pthread_kill=true

# Use native wrappers around the default stdio descriptors.
# Options: [true, false], Default: true.

#native.stdio=true

# Enable verbose logging of native extension loading.
# Options: [true, false], Default: false.

#native.verbose=false

# Allow regexp operations to be interuptible from Ruby.
# Options: [true, false], Default: false.

#regexp.interruptible=false


################################################################################
# thread pooling
################################################################################

# The maximum number of seconds to keep alive a pooled fiber thread.
# Default: 60.

#fiber.thread.pool.ttl=60

# The maximum number of threads to allow in the pool.
# Default: 2147483647.

#thread.pool.max=2147483647

# The minimum number of threads to keep alive in the pool.
# Default: 0.

#thread.pool.min=0

# The maximum number of seconds to keep alive an idle thread.
# Default: 60.

#thread.pool.ttl=60


################################################################################
# miscellaneous
################################################################################

# Enable colorized backtraces.
# Options: [true, false], Default: false.

#backtrace.color=false

# Mask .java lines in Ruby backtraces.
# Options: [true, false], Default: false.

#backtrace.mask=false

# Set the style of exception backtraces.
# Options: [normal, raw, full, mri], Default: normal.

#backtrace.style=normal

# Specify the major Java bytecode version.
# Default: 1.8.

#bytecode.version=1.8

# In some cases of classloader conflicts it might help not to delegate first to the parent classloader but to load first from the jruby-classloader.
# Options: [true, false], Default: true.

#classloader.delegate=true

# Generate consistent object hashes across JVMs
# Options: [true, false], Default: false.

#consistent.hashing=false

# Use lightweight Enumerator#next logic when possible.
# Options: [true, false], Default: true.

#enumerator.lightweight=true

# Use JVM coroutines for Fiber.
# Options: [true, false], Default: false.

#fiber.coroutines=false

# Use fcntl rather than flock for File#flock
# Options: [true, false], Default: true.

#file.flock.fcntl=true

# Use a cache of low-valued Fixnum objects.
# Options: [true, false], Default: true.

#fixnum.cache=true

# Values to retrieve from Fixnum cache, in the range -X..(X-1).
# Default: 256.

#fixnum.cache.size=256

# Use a single global lock for requires.
# Options: [true, false], Default: false.

#global.require.lock=false

# Make non-local flow jumps generate backtraces.
# Options: [true, false], Default: false.

#jump.backtrace=false

# Set in-process launching of e.g. system('ruby ...').
# Options: [true, false], Default: false.

#launch.inproc=false

# Set whether JMX management is enabled.
# Options: [true, false], Default: false.

#management.enabled=false

# Do a true process-obliterating native exec for Kernel#exec.
# Options: [true, false], Default: true.

#native.exec=true

# Use native impls for parts of net/protocol.
# Options: [true, false], Default: false.

#native.net.protocol=false

# (DEPRECATED) Prefer IPv4 network stack
# Options: [true, false], Default: false.

#net.preferIPv4=false

# Enable or disable ObjectSpace.each_object.
# Options: [true, false], Default: false.

#objectspace.enabled=false

# Toggle whether to use "packed" arrays for small tuples.
# Options: [true, false], Default: true.

#packed.arrays=true

# Set the preferred JDK-supported random number generator to use.
# Default: NativePRNGNonBlocking.

#preferred.prng=NativePRNGNonBlocking

# Do not unwrap process streams (issue on some recent JVMs).
# Options: [true, false], Default: false.

#process.noUnwrap=false

# Maintain children static scopes to support scope dumping.
# Options: [true, false], Default: false.

#record.lexical.hierarchy=false

# Use reflection for binding methods, not generated bytecode.
# Options: [true, false], Default: false.

#reflected.handles=false

# Before instantiation, stand up a real Java class for every Ruby class.
# Options: [true, false], Default: false.

#reify.classes=false

# Reify FFI memory structures.
# Options: [true, false], Default: false.

#reify.ffi=false

# Log errors during reification (reify.classes=true).
# Options: [true, false], Default: false.

#reify.logErrors=false

# Attempt to expand instance vars into Java fields
# Options: [true, false], Default: true.

#reify.variables=true

# Maximum number of reified instance variable fields
# Default: 50.

#reify.variables.max=50

# Reify variables into a class named after the Ruby class
# Options: [true, false], Default: false.

#reify.variables.name=false

# Enable or disable SipHash for String hash function.
# Options: [true, false], Default: false.

#siphash.enabled=false

# Set the signal used for dumping thread stacks.
# Options: [USR1, USR2, etc], Default: USR2.

#thread.dump.signal=USR2

# Always ensure volatile semantics for instance variables.
# Options: [true, false], Default: false.

#volatile.variables=false


################################################################################
# debugging and logging
################################################################################

# Set whether full traces are enabled (c-call/c-return).
# Options: [true, false], Default: false.

#debug.fullTrace=false

# Log externally-launched processes.
# Options: [true, false], Default: false.

#debug.launch=false

# Log require/load file searches.
# Options: [true, false], Default: false.

#debug.loadService=false

# Log require/load parse+evaluate times.
# Options: [true, false], Default: false.

#debug.loadService.timing=false

# disables JRuby impl script loads and prints parse exceptions
# Options: [true, false], Default: false.

#debug.parser=false

# Print which script is executed by '-S' flag.
# Options: [true, false], Default: false.

#debug.scriptResolution=false

# Dump class + instance var names on first new of Object subclasses.
# Options: [true, false], Default: false.

#dump.variables=false

# Generate backtraces for heavily-used Errno exceptions (EAGAIN).
# Options: [true, false], Default: false.

#errno.backtrace=false

# Log every time an exception backtrace is generated.
# Options: [true, false], Default: false.

#log.backtraces=false

# Log every time a Kernel#caller backtrace is generated.
# Options: [true, false], Default: false.

#log.callers=false

# Log every time an exception is constructed.
# Options: [true, false], Default: false.

#log.exceptions=false

# Log every time a singleton class is created.
# Options: [true, false], Default: false.

#log.singletons=false

# Log a stack trace every time a singleton class is created.
# Options: [true, false], Default: false.

#log.singletons.verbose=false

# Log every time a built-in warning backtrace is generated.
# Options: [true, false], Default: false.

#log.warnings=false

# Use specified class for logging.
# Options: [class name], Default: org.jruby.util.log.StandardErrorLogger.

#logger.class=org.jruby.util.log.StandardErrorLogger

# Rewrite stack traces from exceptions raised in Java calls.
# Options: [true, false], Default: true.

#rewrite.java.trace=true

# Generate backtraces for heavily-used Errno exceptions (EAGAIN).
# Options: [true, false], Default: false.

#stop_iteration.backtrace=false


################################################################################
# java integration
################################################################################

# Look for .class before .rb to load AOT-compiled code
# Options: [true, false], Default: false.

#aot.loadClasses=false

# Use java.lang.reflect.Proxy for interface impl.
# Options: [true, false], Default: false.

#interfaces.useProxy=false

# Use generated handles instead of reflection for calling Java.
# Options: [true, false], Default: false.

#java.handles=false

# Toggle verbose reporting of all ambiguous calls to Java objects
# Options: [true, false], Default: false.

#ji.ambiguous.calls.debug=false

# Load Java support (class extensions) lazily on demand or ahead of time.
# Options: [true, false], Default: true.

#ji.load.lazy=true

# Log whether setAccessible is working.
# Options: [true, false], Default: false.

#ji.logCanSetAccessible=false

# Extend Java classes without using a proxy object.
# Options: [true, false], Default: false.

#ji.newStyleExtension=false

# Cache Java object wrappers between calls.
# Options: [true, false], Default: false.

#ji.objectProxyCache=false

# Allow external envs to replace JI proxy class factory

#ji.proxyClassFactory=

# Try to set inaccessible Java methods to be accessible.
# Options: [true, false], Default: true.

#ji.setAccessible=true

# Allow Capitalized Java package names.
# Options: [true, false], Default: false.

#ji.upper.case.package.name.allowed=false


################################################################################
# profiling
################################################################################

# Maximum number of methods to consider for profiling.
# Default: 100000.

#profile.max.methods=100000


################################################################################
# command line options
################################################################################

# Wrap execution with a gets() loop. Same as -n.
# Options: [true, false], Default: false.

#cli.assume.loop=false

# Print $_ after each execution of script. Same as -p.
# Options: [true, false], Default: false.

#cli.assume.print=false

# Split $_ into $F for -p or -n. Same as -a.
# Options: [true, false], Default: false.

#cli.autosplit=false

# Set autosplit separator. Same as -F.

#cli.autosplit.separator=

# Backup extension for in-place ARGV files. Same as -i.

#cli.backup.extension=

# Print target script bytecode to stderr. Same as --bytecode.
# Options: [true, false], Default: false.

#cli.bytecode=false

# Check syntax of target script. Same as -c but runs script.
# Options: [true, false], Default: false.

#cli.check.syntax=false

# Print copyright to stderr. Same as --copyright but runs script.
# Options: [true, false], Default: false.

#cli.copyright=false

# Enable debug mode logging. Same as -d.
# Options: [true, false], Default: false.

#cli.debug=false

# Enable/disable did_you_mean.
# Options: [true, false], Default: true.

#cli.did_you_mean.enable=true

# Encoding name to treat external data.

#cli.encoding.external=

# Encoding name to use internally.

#cli.encoding.internal=

# Encoding name to treat source code.

#cli.encoding.source=

# Print command-line usage. Same as --help but runs script.
# Options: [true, false], Default: false.

#cli.help=false

# Set kcode character set. Same as -K (1.8).
# Options: [NIL, NONE, UTF8, SJIS, EUC], Default: NONE.

#cli.kcode=NONE

# Load a bundler Gemfile in cwd before running. Same as -G.
# Options: [true, false], Default: false.

#cli.load.gemfile=false

# Enable parser debug logging. Same as -y.
# Options: [true, false], Default: false.

#cli.parser.debug=false

# Enable line ending processing. Same as -l.
# Options: [true, false], Default: false.

#cli.process.line.ends=false

# Enable instrumented profiling modes.
# Options: [OFF, API, FLAT, GRAPH, HTML, JSON, SERVICE], Default: OFF.

#cli.profiling.mode=OFF

# Profiling service class to use.

#cli.profiling.service=

# Print config properties. Same as --properties but runs script.
# Options: [true, false], Default: false.

#cli.properties=false

# Default record separator.
# Default: "\n".

#cli.record.separator="\n"

# Enable/disable RubyGems.
# Options: [true, false], Default: true.

#cli.rubygems.enable=true

# Enable/disable RUBYOPT processing at start.
# Options: [true, false], Default: true.

#cli.rubyopt.enable=true

# Strip text before shebang in script. Same as -x.
# Options: [true, false], Default: false.

#cli.strip.header=false

# Verbose mode, as -w or -W2. Sets default for cli.warning.level.
# Options: [true, false], Default: false.

#cli.verbose=false

# Print version to stderr. Same as --version.
# Options: [true, false], Default: false.

#cli.version=false

# Warning level (off=0,normal=1,on=2). Same as -W.
# Options: [NIL, FALSE, TRUE], Default: FALSE.

#cli.warning.level=FALSE
```