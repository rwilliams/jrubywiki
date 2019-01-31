JRuby Internal Design
=====================

This page is a quick writeup for those interested. The details here will certainly change over time, but this reflects the state of the world as of JRuby 1.6.

Parsing/lexing
--------------

* JRuby has a YACC/BISON-based parser, basically Ruby's parser ported to Java and using the Jay parser generated.
  * We ship two parsers, in fact, one for evaluating Ruby and one for Ripper.
* The lexer is hand-written, as in Ruby.
* The resulting AST has positioning information for start/end lines. We forked off the JRubyParser project to handle the needs of IDE developers, which additionally stores byte offsets and comments.

Runtime layout
--------------

* The "Ruby" class is the main entry point for the runtime. All objects created have a reference to the runtime, and do not / will not migrate across runtimes without marshalling.
* Every thread using a given runtime is assigned a *ThreadContext* object, stored in a *threadlocal*. We minimize *threadlocal* lookup by frequently passing the *ThreadContext* on the call stack as a parameter.  *ThreadContext* also has fields to *Ruby*, tru(e), fals(e), and nil making it a common default param over passing *Ruby*.
* The *ThreadContext* contains thread-local and call-stack information for the thread, like our artificial call frames and variable scopes, throw/catch markers, a reference to the current in-flight exception, and so on. It also governs logic for generating backtraces.
* Interpreted Ruby calls always push/pop a *Backtrace* on a stack in *ThreadContext*. Compiled calls do not, storing all backtrace information necessary in their compiled Java class/method name. Backtrace generation, then, is a matter of walking the Java backtrace, mining out interpreted and compiled Ruby markers, and aggregating them.

More on threading
-----------------

* Ruby threads are mapped 1:1 to JRuby threads, except in thread pooling mode (experimental) where a pool of threads is maintained from which to fuel Ruby threads. The pooling was added to offset the overhead of APIs that spin up many threads, assuming they're green/lightweight.
* Java threads entering the system from outside the runtime (e.g. started by external Java code) are "adopted" and assigned a *ThreadContext* and *RubyThread* object.
* Ruby's unsafe thread operations (criticalization of a single thread, *Thread#kill*, *Thread#raise*) are supported, but we make different guarantees about them.
  * Criticalization is implemented with a shared global mutex, only protecting critical sections from concurrent execution. It does not stop other threads that are not attempting to run critical section.
  * kill and raise both require that the target thread eventually reach a thread checkpoint, or in pthread parlance a *cancellation point*. They work by sending "mail" to the target thread, which must eventually pick up that mail and propagate the event. Mail is checked periodically (at various language constructs, in many core methods, and every so many calls) by calling *ThreadContext.pollThreadEvents*.

IR - Internal Representation
----------------------------

AST is converted into IRScopes containing IR instrs and operands.  These scopes are interpreted and compiler passes are run over them.  Once it is decided that a scope should be compiled it is passed off to the bytecode generation piece of IR.

* IRSCopes of Ruby: methods (IRMethod), blocks (IRClosure), evals (IREvalScript), class/module body (IRClass/IRModule), script body (IRScriptBody), and for (IRFor).  Note: for is not really an execution context but merely a convenience for analysis (e.g. we don't run an interpreter against instrs in IRFor).
* In general, files loaded via load or require are interpreted at first, allowing only methods called many times to JIT (Just-In-Time) compile to JVM bytecode.  This typically means things like script/module/method bodies never will JIT although for things like the main script file we do AOT.

Methods and dynamic dispatch
----------------------------

* Java-based methods throughout the core classes are annotated with the *JRubyMethod* annotation. This is used by the binding logic to generate (ahead-of-time or at runtime) stub classes called "Invokers" that become our handles to the target method.
* There are two levels of method caching in JRuby: a per-class method cache and per-callsite inline caches.
  * Each class maintains a *generation number* that is monotonically incremented whenever that class or any of its ancestor classes or modules has its method table modified. This is the primary cache-busting mechanism in the system.
  * Each class has in addition to its hashmap-based method table an additional map from method names to *CacheEntry* tuples. *CacheEntry* is an immutable structure referencing a method object and the current class's *generation number* at the time of lookup.
  * All call sites in the system, whether interpreted or compiled, also reference a *CallSite* object...usually some subclass of *CachingCallSite*. *CachingCallSite* is a monomorphic inline cache that holds the most recent *CacheEntry*, using its aggregated method object for dispatch and its generation number for invalidation.
* JRuby 1.6 also ships with an experimental *dynopt* mode which uses the most recent *CacheEntry* from interpreted call sites to insert an additional *static* invocation into compiled code alongside normal dynamic dispatch. This can help some small benchmarks immensely by eliminating all dynamic dispatch. It is not enabled by default in 1.6, but will continue to be researched for future versions.

Compiler
--------

* The compiler's AOT mode compiles one *.rb* file to one *.class* file. The resulting class is simply a "bag of functions", with all methods static. When the methods are actually bound to specific names, the same *Invoker*-generation logic is used to bind them.
* Blocks and class bodies are compiled in the same way; one method in the resulting class per closure.
* The compiler's JIT mode watches for interpreted calls to exceed a simple threshold. The current threshold is 50 calls. It then attempts to compile it, permanently using the compiled code if it succeeds or reverting to interpreted if it fails. There is currently no deoptimization as of JRuby 9.2.x; we never go back to interpreter once we JIT.
* Other than minor static optimizations like simplifying non-expression multiple assignment, eliminating heap-based scopes when not necessary, and avoiding artificial frame pushes when the frame data is not needed, dead code elimination, and less common ones (like preventing to_proc calls on block pass arguments if they are not called within that execution scope).  These are mostly done via compiler passes.
* The resulting bytecode does not decompile to anything very Java-like. As a result, Java decompilers will either produce garbage or produce code that's largely unintelligible.

Future areas for improvement
----------------------------

* When classes are generated at runtime, they are each wrapped with a classloader to allow them to be garbage collected. Super gross, but no other way to safely generate arbitrary numbers of classes without exceeding permgen. We need a ''dynamic method'' or ''anonymous method'' concept, or even a super lightweight class/classloader that would support the same semantics.
* The other big performance pain is representing scope as an external data structure rather than as local variables. This is primarily to support closures and eval, which need to capture variables in scope. We are forced to store all *potentially* closed-over variables, regardless of apparent access, due to Ruby's supporting *Proc* objects used as bindings for *eval*.
* The current compiler does do optimizations but they are all conservative (lacking deoptimization currently) and they do not extend beyond call boundaries (except for looking into contained child closures for stuff like LVA).  
* A method inliner is present via -Xir.inliner but it is not enabled by default.  It also only considers fcalls which are passed blocks.  As time moves forward other call types will be accepted.  
* Deoptimization algorithm has been planned but has not yet been implemented.  
* Native to ruby replacement for the inliner is in its infancy (Integer#times (Java initially -replaced-with-> Ruby). This will allow our inliner to get all the benefits of inlining while not requiring us to make all our core methods ruby implementations (and the startup time associated with interpreting them).
* Profiler support to convert from a simple counting threshold for JITting or inlining to something which detects hot code.  This will allow us to JIT less code but spend more effort on the code we do JIT to produce higher performance.