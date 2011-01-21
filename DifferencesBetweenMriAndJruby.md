Differences Between MRI And JRuby
=================================
Although ideally [MRI](http://en.wikipedia.org/wiki/Ruby_MRI) and JRuby would behave 100% the same in all situations, there are some minor differences. Some differences are due to bugs, and those are not reported here. This page is for differences that are not bugs.

****

<h2 id="native-c-extensions">Native C Extensions</h2>
JRuby cannot run native C extensions.  Popular libraries have all generally been ported to Java Native Extensions.  Also, now that [FFI](http://kenai.com/projects/ruby-ffi/pages/Home) has become a popular alternative to binding to C libraries, using it obviates the need to write a large chunk of native extensions.

<h2 id="continuations">Continuations</h2>
JRuby does not support [continuations](http://www.ruby-doc.org/docs/ProgrammingRuby/html/ref_c_continuation.html) ([Kernel.callcc](http://www.ruby-doc.org/docs/ProgrammingRuby/html/ref_m_kernel.html#Kernel.callcc)).

<h2 id="invoking-external-processes">Invoking external processes</h2>
On Microsoft Windows, JRuby is a little smarter when launching external processes. If the executable file is not a binary executable (`.exe`), MRI requires you give the file suffix as well, but JRuby manages without it.

For example, say you have file `foo.bat` on your PATH and want to run it. 

    system( 'foo' ) # works on JRuby, fails on MRI
    system( 'foo.bat' ) # works both in JRuby and MRI

<h2 id="fork-is-not-implemented">Fork is not implemented</h2>
JRuby doesn't implement `fork()` on any platform, including those where `fork()` is available in MRI.

<h2 id="native-endian-is-big-endian">Native Endian is Big Endian</h2>
Since the JVM presents a _compatible_ CPU to JRuby, the _native_ endianness of JRuby is Big Endian. This does matter for operations that depend on that behavior, like `String#unpack` and `Array#pack` for formats like `I`, `i`, `S`, and `s`.

<h2 id="time-precision">Time precision</h2>
Since it is not possible to obtain `usec` precision under a JVM, `Time.now.usec` cannot return values with microsecond precision.

*Example:*

    > Time.now.usec
    => 582000

Keep this in mind when counting on `usec` precision in your code.

<h2 id="regular-expressions">Regular expressions</h2>
JRuby only has one regular expression engine, which matches Onigurama's behavior. It is not changed in --1.8 mode, so code depending on regular expressions behaving precisely as on MRI 1.8.n may fail on JRuby in --1.8 mode. For instance:

    ruby-1.8.7-p302 > "a".match(/^(.*)+$/)[1]
    => "a"

    jruby-head > "a".match(/^(.*)+$/)[1]
    => ""

<h2 id="thread-priority">Thread priority</h2>
In MRI, the Thread priority can be set to any value in Fixnum (if native threads are enabled) or -3..3 (if not). The default value is 0.

In JRuby, Threads are backed by Java threads, and the priority ranges from 1 to 10, with a default of 5. If you pass a value outside of this range to `Thread#priority=`, the priority will be set to 1 or 10.

(See http://bugs.jruby.org/5289 and http://bugs.jruby.org/5290.)