JRuby usually runs in a mixed mode where some code is being interpreted and some code is converted into JVM bytecode for the JVM to optimize. In order to produce a Ruby trace that only contains lines interesting to a Ruby user, we reconstruct that trace using a combination of JVM stack trace frames and our per-thread, artificially-maintained interpreter frames.

Within the JVM trace, method calls `INTERPRET_ROOT`, `INTERPRET_METHOD`, `INTERPRET_BLOCK`, `INTERPRET_EVAL` and a few others say to the backtrace-builder "pop and use an interpreter frame here" which we maintain on a thread-local object. Certain core methods are mapped internally to Ruby names, like each19 or op_aref. Backtrace-building walks JVM trace backward, including only non-JRuby code, JRuby methods known to map to Ruby core methods, and already-jitted Ruby frames.

Ruby code:

```ruby
def foo
  1.times { bar }
end
def bar
  eval 'raise'
end
foo
```

Ruby trace (same for interpreted and compiled):

```
jruby-1.7.12 ~/projects/discourse $ jruby -X-C -e "def foo; 1.times { bar }; end; def bar; eval 'raise'; end; foo"
RuntimeError: No current exception
     bar at (eval):1
    eval at org/jruby/RubyKernel.java:1101
     bar at -e:1
     foo at -e:1
   times at org/jruby/RubyFixnum.java:275
     foo at -e:1
  (root) at -e:1
```

JVM (raw) trace (interpreted):

```
jruby-1.7.12 ~/projects/discourse $ jruby -X-C -Xbacktrace.style=raw -e "def foo; 1.times { bar }; end; def bar; eval 'raise'; end; foo"
RuntimeError: No current exception
     getStackTrace at java/lang/Thread.java:1568
  getBacktraceData at org/jruby/runtime/backtrace/TraceType.java:175
      getBacktrace at org/jruby/runtime/backtrace/TraceType.java:39
  prepareBacktrace at org/jruby/RubyException.java:224
          preRaise at org/jruby/exceptions/RaiseException.java:213
          preRaise at org/jruby/exceptions/RaiseException.java:194
            <init> at org/jruby/exceptions/RaiseException.java:110
             raise at org/jruby/RubyKernel.java:992
           rbRaise at org/jruby/java/addons/KernelJavaAddons.java:44
              call at org/jruby/internal/runtime/methods/DynamicMethod.java:202
              call at org/jruby/internal/runtime/methods/DynamicMethod.java:198
      cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:306
              call at org/jruby/runtime/callsite/CachingCallSite.java:136
         interpret at org/jruby/ast/VCallNode.java:88
         interpret at org/jruby/ast/NewlineNode.java:105
         interpret at org/jruby/ast/RootNode.java:129
    INTERPRET_EVAL at org/jruby/evaluator/ASTInterpreter.java:95
   evalWithBinding at org/jruby/evaluator/ASTInterpreter.java:184
        evalCommon at org/jruby/RubyKernel.java:1138
            eval19 at org/jruby/RubyKernel.java:1101
              call at org/jruby/internal/runtime/methods/DynamicMethod.java:210
              call at org/jruby/internal/runtime/methods/DynamicMethod.java:206
      cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:326
              call at org/jruby/runtime/callsite/CachingCallSite.java:170
         interpret at org/jruby/ast/FCallOneArgNode.java:36
         interpret at org/jruby/ast/NewlineNode.java:105
  INTERPRET_METHOD at org/jruby/evaluator/ASTInterpreter.java:74
              call at org/jruby/internal/runtime/methods/InterpretedMethod.java:139
      cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:306
              call at org/jruby/runtime/callsite/CachingCallSite.java:136
         interpret at org/jruby/ast/VCallNode.java:88
         interpret at org/jruby/ast/NewlineNode.java:105
   INTERPRET_BLOCK at org/jruby/evaluator/ASTInterpreter.java:112
     evalBlockBody at org/jruby/runtime/Interpreted19Block.java:206
             yield at org/jruby/runtime/Interpreted19Block.java:157
     yieldSpecific at org/jruby/runtime/Interpreted19Block.java:130
     yieldSpecific at org/jruby/runtime/Block.java:111
             times at org/jruby/RubyFixnum.java:275
      cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:316
         callBlock at org/jruby/runtime/callsite/CachingCallSite.java:145
          callIter at org/jruby/runtime/callsite/CachingCallSite.java:154
         interpret at org/jruby/ast/CallNoArgBlockNode.java:64
         interpret at org/jruby/ast/NewlineNode.java:105
  INTERPRET_METHOD at org/jruby/evaluator/ASTInterpreter.java:74
              call at org/jruby/internal/runtime/methods/InterpretedMethod.java:139
      cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:306
              call at org/jruby/runtime/callsite/CachingCallSite.java:136
         interpret at org/jruby/ast/VCallNode.java:88
         interpret at org/jruby/ast/NewlineNode.java:105
         interpret at org/jruby/ast/BlockNode.java:71
         interpret at org/jruby/ast/RootNode.java:129
    INTERPRET_ROOT at org/jruby/evaluator/ASTInterpreter.java:121
    runInterpreter at org/jruby/Ruby.java:838
    runInterpreter at org/jruby/Ruby.java:846
       runNormally at org/jruby/Ruby.java:677
       runFromMain at org/jruby/Ruby.java:522
     doRunFromMain at org/jruby/Main.java:395
       internalRun at org/jruby/Main.java:290
               run at org/jruby/Main.java:217
              main at org/jruby/Main.java:197

```

JVM (raw) trace (compiled at command line):

```

jruby-1.7.12 ~/projects/discourse $ jruby -Xbacktrace.style=raw -e "def foo; 1.times { bar }; end; def bar; eval 'raise'; end; foo"
RuntimeError: No current exception
          getStackTrace at java/lang/Thread.java:1568
       getBacktraceData at org/jruby/runtime/backtrace/TraceType.java:175
           getBacktrace at org/jruby/runtime/backtrace/TraceType.java:39
       prepareBacktrace at org/jruby/RubyException.java:224
               preRaise at org/jruby/exceptions/RaiseException.java:213
               preRaise at org/jruby/exceptions/RaiseException.java:194
                 <init> at org/jruby/exceptions/RaiseException.java:110
                  raise at org/jruby/RubyKernel.java:992
                rbRaise at org/jruby/java/addons/KernelJavaAddons.java:44
                   call at org/jruby/internal/runtime/methods/DynamicMethod.java:202
                   call at org/jruby/internal/runtime/methods/DynamicMethod.java:198
           cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:306
                   call at org/jruby/runtime/callsite/CachingCallSite.java:136
              interpret at org/jruby/ast/VCallNode.java:88
              interpret at org/jruby/ast/NewlineNode.java:105
              interpret at org/jruby/ast/RootNode.java:129
         INTERPRET_EVAL at org/jruby/evaluator/ASTInterpreter.java:95
        evalWithBinding at org/jruby/evaluator/ASTInterpreter.java:184
             evalCommon at org/jruby/RubyKernel.java:1138
                 eval19 at org/jruby/RubyKernel.java:1101
                   call at org/jruby/internal/runtime/methods/DynamicMethod.java:210
                   call at org/jruby/internal/runtime/methods/DynamicMethod.java:206
           cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:326
                   call at org/jruby/runtime/callsite/CachingCallSite.java:170
                    bar at -e:1
                    bar at ruby/-e:1
           cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:306
                   call at org/jruby/runtime/callsite/CachingCallSite.java:136
                    foo at -e:1
                    foo at ruby/-e:1
  yieldSpecificInternal at org/jruby/runtime/CompiledBlock19.java:117
          yieldSpecific at org/jruby/runtime/CompiledBlock19.java:92
          yieldSpecific at org/jruby/runtime/Block.java:111
                  times at org/jruby/RubyFixnum.java:275
           cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:316
              callBlock at org/jruby/runtime/callsite/CachingCallSite.java:145
               callIter at org/jruby/runtime/callsite/CachingCallSite.java:154
                    foo at -e:1
                    foo at ruby/-e:1
           cacheAndCall at org/jruby/runtime/callsite/CachingCallSite.java:306
                   call at org/jruby/runtime/callsite/CachingCallSite.java:136
                 (root) at -e:1
                 (root) at ruby/-e:1
              runScript at org/jruby/Ruby.java:811
              runScript at org/jruby/Ruby.java:804
            runNormally at org/jruby/Ruby.java:673
            runFromMain at org/jruby/Ruby.java:522
          doRunFromMain at org/jruby/Main.java:395
            internalRun at org/jruby/Main.java:290
                    run at org/jruby/Main.java:217
                   main at org/jruby/Main.java:197
```