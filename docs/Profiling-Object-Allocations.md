Need to track how many allocations are happening for a particular class? Would you like to track that allocation back to file and line number? You are in luck. The JVM provides a tool for accomplishing this task.

It will be very slow, but Sun/Oracle/OpenJDK has an object allocation
profiler built in. Examples:

```
jruby -J-Xrunhprof ... # run with object allocation profiling, tracking allocation call stacks 5 deep
jruby -J-Xrunhprof:depth=10 ... # ... call stacks 10 deep
```

Running java -Xrunhprof:help will provide full help. It will be *slow* but you don't need to run a long benchmark; you're looking for aggregate numbers of objects for some number of iterations which doesn't change drastically over a long run. (There's also a full instrumented performance profiler in "runhprof", but it's *dead* slow, because it instruments every method in the system, including every JDK class.)

The object allocation profiles will be dumped to a gigantic log file called java.hprof.txt. The primary interesting bit of information is at the end...a list of object types allocated plus numeric references to the stack trace where they're allocated. Another example:

Command:

```
$ jruby -J-Xrunhprof:depth=20 -e "1_000.times { 'foo' + 'bar' }"
```
A few elements of interest:

```
  12  0.39% 14.11%     32000 1000     32000  1000 336225 org.jruby.RubyString
  13  0.39% 14.51%     32000 1000     32000  1000 336226 org.jruby.RubyString
  14  0.39% 14.90%     31968  999     31968   999 336239 org.jruby.RubyString
```
(Most of the top 20 allocated elements are byte[], which doesn't tell you much)

Note that almost 3000 RubyString objects have been allocated. 1000 of those are 'foo', 1000 of them are 'bar', and 999 of them are the combined string. It's not exactly accurate because the script terminates before the final allocation gets recorded.

The last number before the class name is the allocation trace. Looking at 336225 (search in the same file):

```
TRACE 336225:
       org.jruby.RubyBasicObject.<init>(RubyBasicObject.java:219)
       org.jruby.RubyObject.<init>(RubyObject.java:94)
       org.jruby.RubyString.<init>(RubyString.java:396)
       org.jruby.RubyString.<init>(RubyString.java:424)
       org.jruby.RubyString.newStringShared(RubyString.java:522)
       org.jruby.ast.executable.RuntimeCache.getString(RuntimeCache.java:105)
       org.jruby.ast.executable.AbstractScript.getString0(AbstractScript.java:165)
       ruby.__dash_e__.block_0$RUBY$__file__(-e:1)
       ruby$__dash_e__$block_0$RUBY$__file__.call(ruby$__dash_e__$block_0$RUBY$__file__:65535)
       org.jruby.runtime.CompiledBlock.yield(CompiledBlock.java:112)
       org.jruby.runtime.CompiledBlock.yield(CompiledBlock.java:95)
       org.jruby.runtime.Block.yield(Block.java:130)
       org.jruby.RubyFixnum.times(RubyFixnum.java:261)
       org.jruby.RubyFixnum$INVOKER$i$0$0$times.call(RubyFixnum$INVOKER$i$0$0$times.gen:65535)
       org.jruby.runtime.callsite.CachingCallSite.cacheAndCall(CachingCallSite.java:302)
       org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:144)
       org.jruby.runtime.callsite.CachingCallSite.callIter(CachingCallSite.java:153)
       ruby.__dash_e__.__file__(-e:1)
       ruby.__dash_e__.load(-e:Unknown line)
       org.jruby.Ruby.runScript(Ruby.java:731)
```

This takes us all the way back to runScript in org.jruby.Ruby. The -e:1 lines are our inline script. RubyFixnum.times is the "times" call. And the top seven lines are the parts of JRuby called to allocate a new literal String. Looking at the 336239 trace...
```
TRACE 336239:
       org.jruby.RubyBasicObject.<init>(RubyBasicObject.java:219)
       org.jruby.RubyObject.<init>(RubyObject.java:94)
       org.jruby.RubyString.<init>(RubyString.java:396)
       org.jruby.RubyString.newString(RubyString.java:487)
       org.jruby.RubyString.op_plus(RubyString.java:1039)
       org.jruby.RubyString$INVOKER$i$1$0$op_plus.call(RubyString$INVOKER$i$1$0$op_plus.gen:65535)
       org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:167)
       ruby.__dash_e__.block_0$RUBY$__file__(-e:1)
       ruby$__dash_e__$block_0$RUBY$__file__.call(ruby$__dash_e__$block_0$RUBY$__file__:65535)
       org.jruby.runtime.CompiledBlock.yield(CompiledBlock.java:112)
       org.jruby.runtime.CompiledBlock.yield(CompiledBlock.java:95)
       org.jruby.runtime.Block.yield(Block.java:130)
       org.jruby.RubyFixnum.times(RubyFixnum.java:261)
       org.jruby.RubyFixnum$INVOKER$i$0$0$times.call(RubyFixnum$INVOKER$i$0$0$times.gen:65535)
       org.jruby.runtime.callsite.CachingCallSite.cacheAndCall(CachingCallSite.java:302)
       org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:144)
       org.jruby.runtime.callsite.CachingCallSite.callIter(CachingCallSite.java:153)
       ruby.__dash_e__.__file__(-e:1)
       ruby.__dash_e__.load(-e:Unknown line)
       org.jruby.Ruby.runScript(Ruby.java:731)
```
This is the String#plus call, org.jruby.RubyString.op_plus.

Incidentally, this is also where "class reification" comes in handy:
```
$ jruby -J-Xrunhprof:depth=20 -Xreify.classes=true -e "class Foo; end; 1_000.times { Foo.new }"
...

  17  0.29% 15.50%     23616  984     23616   984 337103 rubyobj.Foo
```

```
TRACE 337103:
       org.jruby.RubyBasicObject.<init>(RubyBasicObject.java:219)
       org.jruby.RubyObject.<init>(RubyObject.java:94)
       rubyobj.Foo.<init>(<Unknown Source>:Unknown line)
       sun.reflect.GeneratedConstructorAccessor4.newInstance(<Unknown Source>:Unknown line)
       sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:27)
       java.lang.reflect.Constructor.newInstance(Constructor.java:513)
       java.lang.Class.newInstance0(Class.java:355)
       java.lang.Class.newInstance(Class.java:308)
       org.jruby.RubyClass$3.allocate(RubyClass.java:142)
       org.jruby.RubyClass.allocate(RubyClass.java:223)
       org.jruby.RubyClass.newInstance(RubyClass.java:809)
       org.jruby.RubyClass$INVOKER$i$newInstance.call(RubyClass$INVOKER$i$newInstance.gen:65535)
       org.jruby.internal.runtime.methods.JavaMethod$JavaMethodZeroOrNBlock.call(JavaMethod.java:256)
       org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:133)
       ruby.__dash_e__.block_0$RUBY$__file__(-e:1)
       ruby$__dash_e__$block_0$RUBY$__file__.call(ruby$__dash_e__$block_0$RUBY$__file__:65535)
       org.jruby.runtime.CompiledBlock.yield(CompiledBlock.java:112)
       org.jruby.runtime.CompiledBlock.yield(CompiledBlock.java:95)
       org.jruby.runtime.Block.yield(Block.java:130)
       org.jruby.RubyFixnum.times(RubyFixnum.java:261)
```

runhprof is a very simple way to get allocation info. It can be used to trim allocation overhead.
