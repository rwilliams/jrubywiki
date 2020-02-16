Although ideally [MRI](http://en.wikipedia.org/wiki/Ruby_MRI) and JRuby would behave 100% the same in all situations, there are some minor differences. Some differences are due to bugs, and those are not reported here. This page is for differences that are not bugs.

****

Stack Trace Differences
-----------------------

Because JRuby is a JVM-based language, you may see some differences in how Ruby stack traces ("backtraces") are rendered. A partial list of such cases is below:

* The "new" frame from constructing an object may not be present due to optimizations.
* Core class methods implemented in Java will show a .java file and line number where CRuby would just show the calling .rb line twice. This should not affect `caller` and `caller_locations` output.
* There may be more or fewer frames due to how we reconstruct a stack trace from the JVM stack.
* Nested blocks will not show the level of nesting.
* Top-level frames may show a different names where CRuby shows "<main>".

In general, code should not rely on exception backtraces matching CRuby exactly, but when there's a questionable difference please open a bug. We try to match CRuby as closely as possible.

Native C Extensions
-------------------

JRuby cannot run native C extensions.  Popular libraries have all generally been ported to Java Native Extensions.  Also, now that [FFI](https://github.com/ffi/ffi) has become a popular alternative to binding to C libraries, using it obviates the need to write a large chunk of native extensions.

Continuations and Fibers
------------------------

JRuby does not support [continuations](http://ruby-doc.com/docs/ProgrammingRuby/html/ref_c_continuation.html) ([Kernel.callcc](http://ruby-doc.com/docs/ProgrammingRuby/html/ref_m_kernel.html#Kernel.callcc)).

Fibers (a form of delimited continuation) are supported on JRuby, but each fiber is backed by a native thread. This can lead to resource issues when attempting to use more fibers than the system allows threads in a given process.

Invoking external processes
---------------------------

On Microsoft Windows, JRuby is a little smarter when launching external processes. If the executable file is not a binary executable (`.exe`), MRI requires you give the file suffix as well, but JRuby manages without it.

For example, say you have file `foo.bat` on your PATH and want to run it. 

    system( 'foo' ) # works on JRuby, fails on MRI
    system( 'foo.bat' ) # works both in JRuby and MRI

Fork is not implemented
-----------------------

JRuby doesn't implement `fork()` on any platform, including those where `fork()` is available in MRI. This is due to the fact that most JVMs cannot be safely forked.

Native Endian is Big Endian
---------------------------

Since the JVM presents a _compatible_ CPU to JRuby, the _native_ endianness of JRuby is Big Endian. This does matter for operations that depend on that behavior, like `String#unpack` and `Array#pack` for formats like `I`, `i`, `S`, and `s`.

Time precision
--------------

Since it is not possible to obtain `usec` precision under a JVM, ~~`Time.now.usec` cannot return values with nanosecond precision~~. This is no longer the case under a POSIX platform (since JRuby 9.1).

    irb(main):004:0> Time.now.usec
    => 815414

On Windows a native system call isn't implemented and thus there's the JVM millisecond precision fallback. 
Keep this in mind when counting on `usec` precision in your code.

    > Time.now.usec
    => 582000

Thread priority
---------------

NOTE: from at least as early as JRuby 1.7.6, Ruby thread priorities are mapped to Java thread priorities, so this section isn't accurate -- you can use the same priority for MRI and JRuby.

~~In MRI, the Thread priority can be set to any value in Fixnum (if native threads are enabled) or -3..3 (if not). The default value is 0.~~

In JRuby, Threads are backed by Java threads, and the priority ranges from 1 to 10, with a default of 5. If you pass a value outside of this range to `Thread#priority=`, the priority will be set to 1 or 10.

NOTE: that the JVM might ignore priorities, it really depends on the target OS and with (older) Java you need to pass the extra `-XX:UseThreadPriorities` for them to be used.

SystemStackError
----------------

JRuby is not able to rescue from `SystemStackError`. If your code rely on this, you should rather try to catch a `Java::JavaLang::StackOverflowError`. See [this ticket](https://github.com/jruby/jruby/issues/1099) for further information.

Argumentless 'proc'
-------------------

If you supply ```proc``` with no arguments and the method happens to have been passed a block then Ruby will end up capturing the passed in block.  Some people discover this and it leads to a rare pattern of ```proc.call```.  We do not support this behavior.  The problem is proc as a function is relatively common but it forces us to deoptimize all uses of proc like ```proc { #some code }```.  The recommended work around is to declare the block explicitly using an ampersand ```def foo(&my_proc)...```.  If you want analogue behavior to proc we also allow ```Proc.new``` which will work exactly the same as a bare proc call.