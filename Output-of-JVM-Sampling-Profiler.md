The output below is from running JRuby with the `--sample` flag (or the underlying JVM with the `-Xprof` flag) running a red/black tree benchmark. These results are not the most accurate profile in the world, but they can often find serious problems with a minimum of profile-time overhead.

A few things to note:

* The `Interpreted` section indicates code running in the JVM's interpreter, i.e. code that the JVM has not yet jitted. If you see more samples here than in the `Compiled` section, it's likely that the JVM has decided that method is too troublesome to JIT. That would be a problem to investigate...but otherwise, this section can mostly be ignored.

* The `Compiled` section indicates code that has transitioned to native code via the JVM's JIT. This is where most interesting profile results live. The top items will be the ones to investigate or report to the JRuby team.

* The `Stub` section is for methods in the JVM that appear to be normal Java methods, but which are actually calls out to a native stub or a native snippit of code. Usually you won't care about this section, unless it dominates the sample counts.

* The `native` column in each section indicates the number of samples spent servicing a native call from the method shown. These numbers usually will not be very high and usually will not be important.

Flat profile of 25.21 secs (1948 total ticks): main

  Interpreted + native   Method                        
  3.6%    70  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$insert$15
  2.1%    41  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$delete_fixup$29
  0.8%    16  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$delete$16
  0.5%     9  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$insert_helper$28
  0.4%     8  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$inorder_walk$21
  0.4%     8  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$successor$19
  0.4%     8  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$reverse_inorder_walk$22
  0.3%     6  +     0    org.jruby.RubyArray.each
  0.3%     6  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$right_rotate$27
  0.3%     5  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$search$23
  0.3%     5  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$predecessor$20
  0.2%     4  +     0    org.jruby.RubyFixnum.times
  0.2%     4  +     0    org.jruby.parser.RubyParser.<clinit>
  0.2%     3  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L7_L
  0.2%     3  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$left_rotate$26
  0.1%     2  +     0    org.jruby.java.invokers.MethodInvoker.<init>
  0.1%     2  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L7_L
  0.1%     2  +     0    org.jruby.RubyFixnum$INVOKER$i$1$0$op_lt.call
  0.1%     0  +     2    java.io.UnixFileSystem.getBooleanAttributes0
  0.1%     2  +     0    org.jruby.RubyString.newString
  0.1%     2  +     0    org.jruby.RubyArray.<init>
  0.1%     0  +     2    java.lang.ClassLoader$NativeLibrary.find
  0.1%     2  +     0    org.jruby.Ruby.initCore
  0.1%     1  +     1    java.lang.Class.getDeclaredConstructors0
  0.1%     1  +     0    sun.reflect.generics.repository.MethodRepository.getReturnType
 18.7%   349  +    16    Total interpreted (including elided)

     Compiled + native   Method                        
 26.0%   507  +     0    org.jruby.internal.runtime.methods.AttrReaderMethod.call
  9.0%   174  +     2    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$insert_helper$28
  6.6%   129  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L7_L
  3.7%    72  +     0    org.jruby.internal.runtime.methods.AttrWriterMethod.call
  3.1%    60  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$minimum$17
  2.7%    50  +     2    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$insert$15
  2.1%    40  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$maximum$18
  2.0%    38  +     0    org.jruby.internal.runtime.methods.CompiledIRMethod.call
  1.9%    37  +     0    org.jruby.RubyBasicObject$INVOKER$i$1$0$op_not_equal.call
  1.7%    34  +     0    org.jruby.RubyClass$INVOKER$i$newInstance.call
  1.7%    34  +     0    org.jruby.RubyBasicObject$INVOKER$i$1$0$op_equal.call
  1.5%    27  +     2    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$delete_fixup$29
  1.3%    25  +     1    org.jruby.runtime.callsite.CachingCallSite.call
  1.0%    19  +     1    org.jruby.runtime.Block.yield
  0.9%    17  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$successor$19
  0.7%    14  +     0    org.jruby.RubyModule.searchWithCache
  0.7%    14  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L8_L
  0.7%    13  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$left_rotate$26
  0.6%    11  +     0    $_dot_dot_.rubybench.time.bench_red_black.invokeOther192:nil?
  0.6%     7  +     4    org.jruby.RubyFixnum.times
  0.5%    10  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L7_L
  0.5%    10  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L8_L
  0.5%     9  +     0    java.lang.invoke.LambdaForm$DMH.invokeStatic_L8_L
  0.5%     9  +     0    $_dot_dot_.rubybench.time.bench_red_black.RUBY$method$inorder_walk$21
  0.5%     9  +     0    org.jruby.RubyKernel$INVOKER$s$0$1$rand19.call
 77.7%  1472  +    42    Total compiled (including elided)

         Stub + native   Method                        
  0.8%     0  +    15    java.lang.String.intern
  0.3%     0  +     6    java.lang.Thread.isInterrupted
  0.3%     0  +     5    java.lang.System.arraycopy
  0.2%     0  +     3    java.lang.System.identityHashCode
  0.2%     0  +     3    java.lang.Thread.currentThread
  0.2%     0  +     3    java.lang.Class.isPrimitive
  0.1%     0  +     2    java.lang.Object.getClass
  0.1%     1  +     1    java.security.AccessController.doPrivileged
  0.1%     0  +     2    java.lang.Object.clone
  0.1%     0  +     2    java.util.zip.Inflater.inflateBytes
  0.1%     0  +     1    java.lang.Class.getName0
  0.1%     0  +     1    java.lang.Class.isInterface
  0.1%     0  +     1    java.lang.Class.getEnclosingMethod0
  0.1%     0  +     1    java.lang.Object.hashCode
  0.1%     0  +     1    java.lang.Class.getModifiers
  0.1%     0  +     1    java.lang.Class.isArray
  0.1%     0  +     1    java.io.UnixFileSystem.getBooleanAttributes0
  0.1%     0  +     1    sun.misc.Unsafe.getObjectVolatile
  2.6%     1  +    50    Total stub

  Thread-local ticks:
  0.9%    18             Class loader


Global summary of 25.21 seconds:
100.0%  1962             Received ticks
  0.7%    13             Received GC ticks
 63.4%  1244             Compilation
  0.9%    18             Class loader