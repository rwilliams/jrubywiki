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
* Every thread using a given runtime is assigned a *ThreadContext* object, stored in a *threadlocal*. We minimize *threadlocal* lookup by frequently passing the *ThreadContext* on the call stack as a parameter.
* The *ThreadContext* contains thread-local and call-stack information for the thread, like our artificial call frames and variable scopes, throw/catch markers, a reference to the current in-flight exception, and so on. It also governs logic for generating backtraces.
* Interpreted Ruby calls always push/pop a *Backtrace* on a stack in *ThreadContext*. Compiled calls do not, storing all backtrace information necessary in their compiled Java class/method name. Backtrace generation, then, is a matter of walking the Java backtrace, mining out interpreted and compiled Ruby markers, and aggregating them.

More on threading
-----------------

* Ruby threads are mapped 1:1 to JRuby threads, except in thread pooling mode (experimental) where a pool of threads is maintained from which to fuel Ruby threads. The pooling was added to offset the overhead of APIs that spin up many threads, assuming they're green/lightweight.
* Java threads entering the system from outside the runtime (e.g. started by external Java code) are "adopted" and assigned a *ThreadContext* and *RubyThread* object.
* Ruby's unsafe thread operations (criticalization of a single thread, *Thread#kill*, *Thread#raise*) are supported, but we make different guarantees about them.
  * Criticalization is implemented with a shared global mutex, only protecting critical sections from concurrent execution. It does not stop other threads that are not attempting to run critical section.
  * kill and raise both require that the target thread eventually reach a thread checkpoint, or in pthread parlance a *cancellation point*. They work by sending "mail" to the target thread, which must eventually pick up that mail and propagate the event. Mail is checked periodically (at various language constructs, in many core methods, and every so many calls) by calling *ThreadContext.pollThreadEvents*.

Interpreter
-----------

* Interpretation proceeds by calling *interpret* on the AST *Node* subclasses. As we walk the nodes of the AST, Ruby behavior is expressed. This performs better than using a "big switch" as in earlier versions of JRuby, likely because virtual dispatch is often faster than doing a large switch-based dispatch.
* There's a shared set of AST nodes for 1.8 and 1.9 modes. A few nodes are specific to one version or the other.
* There are five primary entry points into interpretation: executing an interpreted method or block, evaluating a string containing Ruby code, entering a class body, and executing the toplevel of a script.
* In general, files loaded via load or require are interpreted at first, allowing only methods called many times to JIT (Just-In-Time) compile to JVM bytecode.

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
* The compiler's JIT mode watches for interpreted calls to exceed a simple threshold. As of JRuby 1.6, the threshold is 50 calls. It then attempts to compile it, permanently using the compiled code if it succeeds or reverting to interpreted if it fails. There is no deoptimization as of JRuby 1.6; we never go back to interpreter once we JIT.
* Other than minor static optimizations like simplifying non-expression multiple assignment, eliminating heap-based scopes when not necessary, and avoiding artificial frame pushes when the frame data is not needed, the compiler does not perform many optimizations. It is largely a translation of the interpreted logic directly into JVM bytecode.
* The resulting bytecode does not decompile to anything very Java-like. As a result, Java decompilers will either produce garbage or produce code that's largely unintelligible.
* 1.8 and 1.9 modes share most of the same compiler, with 1.9 mode overriding a few key areas like multiple assignment, method/block arguments, and 1.9-specific features like "stabby lamda" (the ->(){} block form).

Compatibility
-------------

* JRuby 1.6 has achieved a very high level of compatibility with Ruby 1.8.7. We run nearly as many [http://rubyspec.org RubySpec] specs as MRI 1.8.7, with differences largely restricted to things the JVM can't do (like fork, true exec, other low-level system functions).
* JRuby 1.6 is also the first release where we are recommend users try 1.9 mode and report issues. Key applications and libraries like RubyGems, Rake, Rails, and RSpec all work properly in 1.9 mode.
* JRuby passes 100% of the Rails test suites, and we maintain a continuous integration build to watch for incompatible changes. In production, "JRuby on Rails" generally performs better than MRI 1.8.7, and usually on par with or better than MRI 1.9.2.
* A number of C-based extensions have been ported to JRuby, such as for database accessor JSON parsing.
* JRuby 1.6 ships experimental C extension support, to aid migration. We do not, however, recommend heavy use of C extensions, since the call boundaries are much slower than Java-based extensions and C extensions are necessarily forced to run single-threaded. There are also deployment concerns; C extensions maintain process-global state, making it impossible to run multiple JRuby instances in the same process.

Future areas for improvement
----------------------------

* When classes are generated at runtime, they are each wrapped with a classloader to allow them to be garbage collected. Super gross, but no other way to safely generate arbitrary numbers of classes without exceeding permgen. We need a ''dynamic method'' or ''anonymous method'' concept, or even a super lightweight class/classloader that would support the same semantics.
* The other big performance pain is representing scope as an external data structure rather than as local variables. This is primarily to support closures and eval, which need to capture variables in scope. We are forced to store all *potentially* closed-over variables, regardless of apparent access, due to Ruby's supporting *Proc* objects used as bindings for *eval*.
* The current compiler does very little to optimize a method as a whole. The minor optimizations performed are at the granularity of AST nodes, eliminating the potential for full-method optimization. We also do very little dynamic optimization and no deoptimization, which means we must be more conservative in the optimizations we perform.