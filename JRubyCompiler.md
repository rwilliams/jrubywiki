The JRuby compiler supports both ahead-of-time (AOT) and just-in-time (JIT) compiling.

Ahead-Of-Time (AOT) Compilation
-------------------------------
The typical way to run the AOT compiler is to run 
```bash
jrubyc <script name>
```
Or, on Microsoft Windows:
```bash
jruby -S jrubyc <script name>
```

This command outputs a `.class` file in the current directory with parent directories and package matching where the file lives. So the following command:
```bash
jrubyc foo/bar/test.rb 
```

will output:
```bash
foo/bar/test.class
```

To run the file produced by the compiler, use the `-cp` (class searchpath) parameter with both the current directory (parent directory for foo/bar/test.class above) and the path to `jruby.jar`, and execute it as you would a normal Java class named foo.bar.test:

```bash
java -cp .:/path/to/jruby.jar foo.bar.test
```

**Note from Dhjdhj 08:17, 6 November 2008 (PST)**

If you're getting errors about missing requires, then you need to use the [jruby-complete jar](http://repository.codehaus.org/org/jruby/jruby-complete) for running. I do not know why this jar file is not included with the standard distribution.

Also, if you're trying to do this from a Microsoft Windows command window (not cygwin), then to compile your code you need to use the command

```bash
jruby -S jrubyc foo/bar/test.rb
```

which outputs `test.class` to the `foo\bar\` subdirectory of the current directory.

Just-In-Time (JIT) Compilation
------------------------------
In addition to AOT compiling described above, JRuby supports using the compiler in JIT mode, where it will attempt to compile methods as they're called. After a call threshold is reached, the JIT tries to compile the method body in question, including any blocks it contains. If the compilation succeeds, the compiled version is used from then on. Otherwise, the method is permanently marked "interpret only".

The JIT is enabled by default with a threshold of 50. You can turn it off by using the `-J-Djruby.jit.enabled=false` flag. For example:
 jruby -J-Djruby.jit.enabled=false myscript.rb

For more information on JIT flags, see [[PerformanceTuning#JIT_Runtime_Properties|JIT Runtime Properties]].

Performance
-----------
Here are a few microbenchmarks comparing Ruby, JRuby interpreted, and JRuby compiled (server VM numbers show worst and best numbers):

		fib(30) Ruby:                          1.67s
    fib(30) JRuby interpreted (client VM): 3.93s
    fib(30) JRuby interpreted (server VM): 2.28s to 2.08s
    fib(30) JRuby compiled (client VM):    1.89s to 1.79s
    fib(30) JRuby compiled (server VM):    1.66s to 0.86s

Tweaking and troubleshooting
----------------------------
There are other interesting properties you can use.

**Note:** To see all the JRuby properties, use the `jruby --properties` command:

* jruby.compile.mode=OFF|JIT|FORCE (default JIT)
Sets compilation to none, JIT, or AOT

* jruby.jit.threshold=## (default 50)
Sets the threshold for methods to get jitted. (I usually use it to make all methods compile before execution, using threshold=0.)

* jruby.jit.logging=true|false (default false)
:Logs each method as it's compiled, so you can see what kind of coverage you're getting or if the methods you want to JIT are getting JITed.

* jruby.jit.logging.verbose=true|false (default false)
:Logs each method that fails to compile, so you can see if compilation problems are keeping methods from JITing.

* jruby.jit.exclude=<class_or_module_name>|<class_or_module_name>#<methodname>|<methodname>
:Excludes either the class, module, instance_method on a class, instance_method on a module, or a global method name.

As of late September 2007, the JRuby compiler is considered complete. Any features missing are to be considered bugs. As part of JRuby's test run, the entire Ruby standard library is compiled to Java bytecode.

There are two known issues with the JRuby compiler at present, which might or might not get fixed:

* Calling a method with a `while false` loop as its parameter will fail to execute

 foo(while false; end) # results in an error

* The `retry` keyword is not currently supported outside a `rescue` block, largely because of performance considerations and general confusion around how it's supposed to work. Additionally, no one appears to use `retry` outside `rescue`.

JRuby Compiler Design
=====================
JRuby compiles Ruby code to Java bytecode. Once complete, there's no interpretation done, except for `eval` calls. Evaluated code never gets compiled; however, if the `eval` defines a method that's called enough, it will also eventually get JIT compiled to bytecode. JRuby is a mixed-mode engine.

Output Is a Class File
----------------------
Given a single input `.rb` file, JRuby produces a single output `.class` file. This was a key design goal for the compiler. Other languages (including Groovy) and other Ruby implementations (including XRuby) produce numerous classes from an input file, in some cases, dozens and dozens of classes if the input file is very large and complex. JRuby produces one `.class` file.

Multiple Passes Over AST
------------------------
JRuby compiles from the same AST as it interprets from. There is a first pass over the AST before compilation to determine certain runtime characteristics:

* Does a method have closures in it?
* Does a method have calls to eval or other scope and frame-aware methods?
* Does a method have class definitions in it?
* Does a method define other methods?
* .... and so on

Based on this pass, the compiler determines scoping characteristics of all code in the method, selectively choosing pure heap-based variables or pure stack-based variables. Only methods and leaf closures without `eval`, `closures`, and so on, can use normal stack-based local variables. Performance is significantly faster with stack variables.

Methods in Output Class File
----------------------------
The resulting class file from JRuby contains at a minimum the following methods to start:

* A normal `main()` method for running from the command line. (It grabs a default JRuby runtime and launches itself.)
* A `load()` instance method that represents a normal top-level loading of the script into a runtime. This performs pre and post script setup and teardown.
* A `run()` instance method that represents a bare execution of the script's contents. This is used by the JIT, where setup and teardown is handled outside the JITed code on a method-by-method basis
* A `__file__()` method that represents the body of the script. This is where script execution eventually starts.

Then, depending on the contents of the file, additional methods are added:

* Normal method definition bodies become Java methods.
* Class and module bodies become Java methods.
* Closure bodies become Java methods.
* Rescue and ensure bodies become synthetic methods.
* If the normal top-level script method is too long, it's split every 500 top-level syntactic elements and chained. (We did run into one large flat file that broke the method size limit.) We do not yet perform chaining on normal method bodies because we have not encountered any that are too large.

Direct Invocation, Binding Into the MOP, and Reflection
-------------------------------------------------------
Only class bodies, `rescue/ensure` bodies, and chained top-level script methods get directly invoked during script execution. The others are bound into the MetaObject Protocol (MOP) at runtime.

Binding occurs in one of two ways:
* By generating a small stub class that implements `DynamicMethod` and invokes the target method on the target script directly
* By doing the same with reflection

In our testing, generating stub _invoker_ classes has always been faster than reflection, especially on older JVMs. For the time being, that's the preferred way to bind methods, but I'm going to get reflection-based binding working again for limited/restricted environments like applets. With reflection-based binding and pre-compiled Ruby code with no `evals`, JIT compilation could be completely turned off and no classes would ever be generated in memory by JRuby.

Walkthrough of a Simple Script
==============================
Here's a walkthrough of a simple script:

* we enter into the script body in the `__file__` method
* require would first look for `.rb` files, then try to load `.class`

```ruby
require 'foo'

# normal code in the method body
puts 'here we go'

# upon encountering a method def, a new method is started in the class
def bar
 # this is a simple method body, and would use stack-based vars
 puts 'hello'
end
# once the method has been compiled, binding code is added to __file__

# class definitions become methods as well, building the class
class MyClass
 # this is code in the body of the class
 puts 'here'

 # a method in the class is compiled like any other method body
 def something(a, b = 2, *c, &block)
	 # this method has all four param types:
	 # normal, optional, "rest" or varargs, and block argument
	 # the compiler generates code to assign these from an incoming
	 # IRubyObject[]

	 # this method has a closure, so it would use heap-based vars
	 # ... but the closure would use stack vars, since it's a simple leaf
	 1.times { puts 'in closure' }
 end
 # method is completed, bound into the class we're building
end
# end of class definition; __file__ code invokes the class body directly 

# any begin block or method body with a rescue/ensure attached will
# be compiled as a synthetic method. This also necessarily means that
# method bodies containing rescue/ensure must be heap-based.
begin
 puts 'rescue me'
rescue
 puts 'rescued!'
ensure
 puts 'ensured!'
end
```

Sample Run of the JRuby Compiler
================================
```bash
~/NetBeansProjects/jruby $ jruby sample_script.rb
here we go
here
rescue me
ensured!

~/NetBeansProjects/jruby $ jrubyc sample_script.rb
Compiling file "sample_script.rb" as class "sample_script"

~/NetBeansProjects/jruby $ ls -l sample_script.*
-rw-r--r--   1 headius  headius  8396 Oct  4 09:38 sample_script.class
-rw-r--r--   1 headius  headius  1449 Oct  4 09:38 sample_script.rb

~/NetBeansProjects/jruby $ export
CLASSPATH=lib/jruby.jar:lib/asm-3.0.jar:lib/jna.jar:.

~/NetBeansProjects/jruby $ java sample_script
here we go
here
rescue me
ensured!
```
