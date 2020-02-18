[[Home|&raquo; JRuby Project Wiki Home Page]]
<h1>Embedding JRuby</h1>
Using Java from Ruby is JRuby's best-known feature---but you can also go in the other direction, and use Ruby from Java.  There are several different ways to do this. You can execute entire Ruby scripts, call individual Ruby methods, or even implement a Java interface in Ruby (thus allowing you to treat Ruby objects like Java ones).  We refer to all these techniques generically as "embedding."  This section will explain how to embed JRuby in your Java project.

__TOC__

= JRuby Embed (originally known as Red Bridge) =

JRuby has long had a private embedding API, which was closely tied to the runtime's internals and therefore changed frequently as JRuby evolved. Since version 1.4, however, we have also provided a more stable public API, known originally as Red Bridge, now known as JRuby Embed. Existing Java programs written to the [[DirectJRubyEmbedding|raw API's]] will probably still continue to work, but we strongly recommend Red Bridge for all new projects.  Note that using Jruby Embed works well for doing eval of ruby code, and/or sharing a few variables back and forth with a ruby script, but if you want anything more (like new'ing up jruby instances within java, etc.) than that you'll need to code to the [[DirectJRubyEmbedding|raw API's]].

== Features of Red Bridge ==
Red Bridge consists of two layers: Embed Core on the bottom, and implementations of [http://www.jcp.org/en/jsr/detail?id=223 JSR223] and [http://jakarta.apache.org/bsf/ BSF] (Bean Scripting Framework) on top. Embed Core is JRuby-specific, and can take advantage of much of JRuby's power. JSR223 and BSF are more general interfaces that provide a common ground across scripting languages.

See also [[JavaIntegration]] for some more JSR223/BSF examples.

Which API should you use? For projects where Ruby is the only scripting language involved, we recommend Embed Core for the following reasons:

# With Embed Core, you can create several Ruby environments in one JVM, and configure them individually (via <code>org.jruby.RubyInstanceConfig</code>. With the other APIs, configuration options can only be set globally, via the <code>System</code> properties.
# Embed Core offers several shortcuts, such as loading scripts from a <code>java.io.InputStream</code>, or returning Java-friendly objects from Ruby code. These allow you to skip a lot of boilerplate.

For projects requiring multiple scripting languages, JSR223 is a good fit. 

Though the API is language-independent, JRuby's implementation of it allows you to set some Ruby-specific options. In particular, you can control the threading model of the scripting engine, the lifetime of local variables, compilation mode, and how line numbers are displayed.

The full [http://jruby.org/apidocs/ API documentation] has all the gory details. It's worth talking about a couple of the finer points here.  Also the old [http://kenai.com/projects/redbridge/pages/Home wiki] has some example code that apparently didn't make it to the migration here.  Here are a list of its advantages:

=== Context Type ===

The context type (singleton, thread-safe, single-threaded, or concurrent) determines how JRuby maintains its internal Ruby state. Choosing the wrong type can lead to surprising results, so it's worth talking about how to choose the right one.

For simple cases where you're only using one Ruby runtime across your Java program, and you don't need to access it from multiple threads, choose the singleton type. If you want to run Ruby scripts in a multi-threaded environment, you should use the thread-safe type instead. A newly added type, concurrent, is a mixture of singleton and thread-safe. The concurrent type has a singleton Ruby runtime and thread local variable map.

=== Variables ===

Variable sharing is one area where RedBridge provides additional power to you. Typical scripting APIs rely on the blunt instrument of global variables to share data. With this approach, you'd set a few globals (remembering to use <code>$</code> at the beginning of the variable name for Ruby), run a script, and then retrieve the results through other globals.

RedBridge allows this use, of course, but offers better choices as well. You can easily share not just global variables, but also local and instance variables, across the Ruby / Java divide. JRuby uses a mechanism called a ''variable map'' to track variable names and values between Ruby and Java. Various options control how this variable map is reused from one invocation to another.

You can also pass data in through method arguments, and work directly with return values. These values are accessible on the Java side as normal Java objects: you're not forced to convert everything to strings or primitive types.

== Download ==

JRuby Embed (Red Bridge) binaries have been included in the official [http://www.jruby.org/download JRuby download] since 1.4.0RC1. Starting with version 1.5, the Red Bridge ''source'' is included in the JRuby core as well. 

JSR 223 and BSF are stable APIs. The Embed Core API is somewhat stable, but may change slightly from release to release.

== Getting Started ==

A simple 'Hello World' example is a great way to get to know Red Bridge.

Before you start programming, make sure the necessary jar archives are in your classpath:

* Embed Core
** <code>jruby-complete.jar</code> or <code>jruby.jar</code>

* JSR223
** <code>jruby-complete.jar</code> or <code>jruby.jar</code>
** <code>script-api.jar</code> (if you are using JDK1.5)

* BSF
** <code>jruby-complete.jar</code> or <code>jruby.jar</code>
** <code>bsf.jar</code>
** <code>commons-logging-[version].jar</code>


Here's how to execute a simple Ruby script (<code>puts "Hello World!"</code>) using the three JRuby Embed APIs:
* Core

```java
package vanilla;

import org.jruby.embed.ScriptingContainer;

public class HelloWorld {

    private HelloWorld() {
        ScriptingContainer container = new ScriptingContainer();
        container.runScriptlet("puts 'Hello World!'");
    }

    public static void main(String[] args) {
        new HelloWorld();
    }
}
```

* JSR223

```java
package redbridge;

import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class Jsr223HelloWorld {

    private Jsr223HelloWorld() throws ScriptException {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        engine.eval("puts 'Hello World!'");
    }

    public static void main(String[] args) throws ScriptException {
        new Jsr223HelloWorld();
    }
}
```

* BSF

```java
package azuki;

import org.apache.bsf.BSFException;
import org.apache.bsf.BSFManager;

public class BsfHelloWorld {
    private BsfHelloWorld() throws BSFException {
        BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
        BSFManager manager = new BSFManager();
        manager.exec("jruby", "<script>", 0, 0, "puts 'Hello World!'");
    }

    public static void main(String[] args) throws BSFException {
        new BsfHelloWorld();
    }

}
```

All three programs print out "Hello World!" to standard output.

== Configurations ==

JRuby Embed (RedBridge) allows you to configure its environment and behavior. In many cases, the default options are all you need. For situations when you want more control, you can use the options described in this section.

With Embed Core, you typically set options through constructor or method arguments. With JSR223, you use <code>System</code> properties instead.

RedBridge supports both per-container ("per-engine," in JSR223 parlance) and per-evaluation options. When you set per-container options using system properties, you must set before instantiating a scripting container or engine. When you set options using methods of Embed Core, you must set before using put/parse/runScriptlet methods. These methods instantiate and initialize Ruby runtime based on the configuration. Once Ruby runtime has been initialized, per-container configuration won't be referred anymore. Per-evaluation options should be set just before running Ruby code.

*Per-container configurations:
**JRuby Home
**Classpath
**Context Instance Type
**Local Variable Behavior
**Compile Mode
**Ruby Version

*Per-evaluation configurations:
**Disabling Sharing Variables
**Line Number

As of JRuby 1.5.0, Red Bridge offers several configuration methods. This helps separate the internal API from the publicly used one. Please see [[JRubyOptions]] for a list of JRuby options supported by Red Bridge.

=== JRuby Home ===

Scope: Per Container<br/>
Propety Name: jruby.home

JRuby Home is is where JRuby looks for its built-in libraries. Internally, JRuby Embed first checks for the <code>JRUBY_HOME</code> environment variable, then the <code>jruby.home</code> system property, then the value embedded in <code>jruby-complete.jar</code>. Using <code>jruby-complete.jar</code> is the simplest approach, because it frees you from having to set this option yourself.

If you'd rather use the environment variable, you can set it from your OS . Here's what that looks like on UNIX and OS X (assuming your shell is <code>bash</code>):

<pre>
export JRUBY_HOME=/Users/yoko/Tools/jruby-1.3.1</pre>

The other alternative is the <code>jruby.home system</code> property. There are two ways to set its value: via the command line, or using <code>java.langSystem#setProperty()</code>.

* Core, JSR223, BSF

```
java –J-Djruby.home=/Users/yoko/Tools/jruby-1.3.1 –cp …
```

```java
System.setProperty("jruby.home", "/Users/yoko/Tools/jruby-1.3.1");
ScriptingContainer container = new ScriptingContainer();
```

With Embed Core, you can also set <code>jruby.home</code> individually for each scripting container instance:

* Core

```java
ScriptingContainer container = new ScriptingContainer();
// JRuby 1.5.x
container.setHomeDirectory("/Users/yoko/Tools/jruby-1.5.6");
// JRuby 1.4.0
//container.getProvider().getRubyInstanceConfig().setJRubyHome("/Users/yoko/Tools/jruby-1.3.1");
```

=== Class Path (Load Path) ===

Scope: Per Container<br/>
Propety Name: org.jruby.embed.class.path (or java.class.path)

Classpaths are used to load Ruby scripts as well as Java classes and jar archives. Embed Core and BSF have a method to set classpath. However, JSR223 doesn't have such method or method argument. Thus, JSR223 need to be set classpath by system property. Either org.jruby.embed.class.path or java.class.path property names can be used to set classpath. The org.jruby.embed.class.path system property is avaiable to use in Embed Core and BSF, too. In case of Embed Core and JSR223, a value assigned to org.jruby.embed.class.path is looked up first, then java.class.path. This means that only org.jruby.embed.class.path is used if exists.  As for BSF, after java.class.path is looked up, org.jruby.embed.class.path  is added to if exists. The format of the paths is the same as Java's class path syntax, :(colon) separated path string on Unix and OS X or ;(semi-colon) separated one on Windows. Be sure to set classpaths before you instantiate javax.script.ScriptEngineManager or register engine to org.apache.bsf.BSFManager. A related topic is in [[ClasspathAndLoadPath]]

Samples below run testMath.rb, which is included in JRuby’s source archive. The script, testMath.rb, needs minirunit.rb, which should be loaded from the classpath.

* Core

```java
package vanilla;

import java.util.ArrayList;
import java.util.List;
import org.jruby.embed.PathType;
import org.jruby.embed.ScriptingContainer;

public class LoadPathSample {
    private final static String jrubyhome = "/Users/yoko/Tools/jruby-1.3.1";
    private final String filename = jrubyhome + "/test/testMath.rb";

    private LoadPathSample() {
        ScriptingContainer container = new ScriptingContainer();
        List<String> loadPaths = new ArrayList();
        loadPaths.add(jrubyhome);
        // JRuby 1.5.x
        container.setLoadPaths(loadPaths);
        // JRuby 1.4.0
        //container.getProvider().setLoadPaths(loadPaths);
        container.runScriptlet(PathType.ABSOLUTE, filename);
    }

    public static void main(String[] args) {
        new LoadPathSample();
    }
}
```


* JSR223

```java
package redbridge;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.Reader;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class Jsr223LoadPathSample {
    private final static String jrubyhome = "/Users/yoko/Tools/jruby-1.3.1";
    private final String filename = jrubyhome + "/test/testMath.rb";

    private Jsr223LoadPathSample() throws ScriptException, FileNotFoundException {
        System.setProperty("org.jruby.embed.class.path", jrubyhome);
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        Reader reader = new FileReader(filename);
        engine.eval(reader);
    }

    public static void main(String[] args) throws ScriptException, FileNotFoundException {
        new Jsr223LoadPathSample();
    }
}
```

* BSF

```java
package azuki;

import java.io.FileNotFoundException;
import org.apache.bsf.BSFException;
import org.apache.bsf.BSFManager;
import org.jruby.embed.PathType;

public class BsfLoadPathSample {
    private final static String jrubyhome = "/Users/yoko/Tools/jruby-1.3.1";
    private final String filename = jrubyhome + "/test/testMath.rb";
    
    private BsfLoadPathSample() throws BSFException, FileNotFoundException {
        BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
        BSFManager manager = new BSFManager();
        manager.setClassPath(jrubyhome);
        manager.exec("jruby", filename, 0, 0, PathType.ABSOLUTE);
    }

    public static void main(String[] args) throws BSFException, FileNotFoundException {
        new BsfLoadPathSample();
    }
}
```

=== Context Instance Type ===

Scope: Per Container<br/>
Property Name: org.jruby.embed.localcontext.scope<br/>
Value: '''singleton'''(default in JRuby 1.6.x/1.5.x/1.4.0 and JRuby trunk; rev. 9e557a2 and later),<br/>
'''theadsafe''' (default in JRuby 1.4.0RC1, RC2, RC3),<br/>
'''singlethread''', <br/>
or '''concurrent''' (since JRuby 1.6.0)

The context instance type is a type of a local context. The local context holds Ruby runtime, name-value pairs for sharing variables between Java and Ruby, default I/O streams (reader/writer/error writer), and attributes. The context is saved in one of four types, threadsafe, singlethread, singleton, or concurrent. Three types are defined in org.jruby.embed.LocalContextScope and also can be set by a system property. 

Make sure to set the context type before you instantiate javax.script.ScriptEngineManager or register engine to org.apache.bsf.BSFManager.

==== Singleton ====
This model uses a well known Singleton pattern, and the only one instance of a local context will exist on JVM. No matter how many ScriptingConatiners (or ScriptEngines (JSR223)) are instantiated, a single Ruby runtime and variable map are there. Please be aware, thread safety is users' responsibility.<br/>

```
+------------------------------------------------------------+
|                       Variable Map                         |
+------------------------------------------------------------+
+------------------------------------------------------------+
|                       Ruby runtime                         |
+------------------------------------------------------------+
+------------------+ +------------------+ +------------------+
|ScriptingContainer| |ScriptingContainer| |ScriptingContainer|
+------------------+ +------------------+ +------------------+
+------------------------------------------------------------+
|                         JVM                                |
+------------------------------------------------------------+

                Singleton Local Context Type
          (Thread safety is users' responsibility)
```

* Core

<pre name="java">
ScriptingContainer instance = new ScriptingContainer(LocalContextScope.SINGLETON);
</pre>

* JSR223/BSF

<pre name="java">
System.setProperty("org.jruby.embed.localcontext.scope", "singleton");
</pre>

==== ThreadSafe ====
Script's parsings and evaluations should be safely performed on a multi-threaded environment such as servlet container. A single ScriptingContainer creates thread local Ruby runtimes and variable maps.<br/>

```
+------------------+ +------------------+ +------------------+
|   Variable Map   | |   Variable Map   | |   Variable Map   |
+------------------+ +------------------+ +------------------+
+------------------+ +------------------+ +------------------+
|   Ruby runtime   | |   Ruby runtime   | |   Ruby runtime   |
+------------------+ +------------------+ +------------------+
+------------------------------------------------------------+
|                     ScriptingContainer                     |
+------------------------------------------------------------+
+------------------+ +------------------+ +------------------+
|   Java Thread    | |   Java Thread    | |   Java Thread    |
+------------------+ +------------------+ +------------------+
+------------------------------------------------------------+
|                         JVM                                |
+------------------------------------------------------------+

                Threadsafe Local Context Type
       (Per Thread Isolated Variable Map and Runtime)
```

* Core

<pre name="java">
ScriptingContainer instance = new ScriptingContainer(LocalContextScope.THREADSAFE);
</pre>

* JSR223/BSF

<pre name="java">
System.setProperty("org.jruby.embed.localcontext.scope", "threadsafe");
</pre>

==== SingleThread ====
This model pretends as if there is only one thread in the world, and does not mind race condition at all. This naive model is used in many other JSR223 ScriptEngines. Users are responsible to thread safety.<br/>

```
+------------------+ +------------------+ +------------------+
|   Variable Map   | |   Variable Map   | |   Variable Map   |
+------------------+ +------------------+ +------------------+
+------------------+ +------------------+ +------------------+
|   Ruby runtime   | |   Ruby runtime   | |   Ruby runtime   |
+------------------+ +------------------+ +------------------+
+------------------+ +------------------+ +------------------+
|ScriptingContainer| |ScriptingContainer| |ScriptingContainer|
+------------------+ +------------------+ +------------------+
+------------------------------------------------------------+
|                         JVM                                |
+------------------------------------------------------------+

                Singlethread Local Context Type
          (Thread safety is users' responsibility)
```

* Core

<pre name="java">
ScriptingContainer instance = new ScriptingContainer(LocalContextScope.SINGLETHREAD);
</pre>

* JSR223/BSF

<pre name="java">
System.setProperty("org.jruby.embed.localcontext.scope", "singlethread");
</pre>


==== Concurrent ====

This model isolates variable maps but shares Ruby runtime. A single ScriptingContainer created only one runtime and thread local variable maps. Global variables and top level variables/constants are not thread safe, but variables/constants tied to receiver objects are thread local.<br/>

```
+------------------+ +------------------+ +------------------+
|   Variable Map   | |   Variable Map   | |   Variable Map   |
+------------------+ +------------------+ +------------------+
+------------------------------------------------------------+
|                        Ruby runtime                        |
+------------------------------------------------------------+
+------------------------------------------------------------+
|                     ScriptingContainer                     |
+------------------------------------------------------------+
+------------------+ +------------------+ +------------------+
|   Java Thread    | |   Java Thread    | |   Java Thread    |
+------------------+ +------------------+ +------------------+
+------------------------------------------------------------+
|                         JVM                                |
+------------------------------------------------------------+

                Concurrent Local Context Type
    (Per Thread Isolated Variable Map and Singleton Runtime)
```

* Core

<pre name="java">
ScriptingContainer instance = new ScriptingContainer(LocalContextScope.CONCURRENT);
</pre>

* JSR223/BSF

<pre name="java">
System.setProperty("org.jruby.embed.localcontext.scope", "concurrent");
</pre>


=== Local Variable Behavior Options ===

Scope: Per Container<br/>
Property Name: org.jruby.embed.localvariable.behavior<br/>
Value: '''transient''' (default for core), '''persistent''', '''global''' (default for JSR223), or '''bsf''' (for BSF)

JRuby Embed enables to share Ruby's local, instance, global variables, and constants. To share these variables between Ruby and Java, JRuby Embed offers four types of local variable behaviors, transient, persistent, global, and bsf. Four types are defined in org.jruby.embed.LocalVariableBehavior and also can be set by a system property.


==== Transient Local Variable Behavior ====

Default for Embed Core. Local variables' scope is faithful to Ruby semantics. This means local variable does not survive over the multiple evaluations. After the each evaluation, local variable will vanish away. However, instance and global variables, and constants survive unless those are removed explicitly. If you use global variables, the variables can be referred literally globally in Ruby runtime and exist as long as the runtime is alive. Be careful to the scope of global variables so that you don’t mix in vulnerabilities in a web application.<br/>

* Core

```java
ScriptingContainer instance = new ScriptingContainer();
```
or
```java
ScriptingContainer instance = new ScriptingContainer(LocalVariableBehavior.TRANSIENT);
```
or
```java
ScriptingContainer instance = new ScriptingContainer(LocalContextScope.SINGLETHREAD, LocalVariableBehavior.TRANSIENT);
```

* JSR223

```java
System.setProperty("org.jruby.embed.localvariable.behavior", "transient");
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("jruby");
```

* BSF
BSF can choose only BSF type.

==== Persistent Local Variable Behavior ====

When this type is chosen, JRuby Embed keeps sharing all local variables' over multiple evaluations. This might not be a semantically correct usage, but is useful in some cases especially for users who have BSF background.<br/>

* Core
```java
ScriptingContainer instance = new ScriptingContainer(LocalVariableBehavior.PERSISTENT);
```
or
```java
ScriptingContainer instance = new ScriptingContainer(LocalContextScope.SINGLETHREAD, LocalVariableBehavior.PERSISTENT);
```

* JSR223
```java
System.setProperty("org.jruby.embed.localvariable.behavior", "persistent");
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("jruby");
```

* BSF
BSF can choose only BSF type.

==== Global Local Variable Behavior ====

Default for JSR223. This behavior might be convenient to users who have used JSR223 reference implementation released at scripging.dev.java.net and don't want change any code at all. With names like Ruby's local variable name, variables are mapped to Ruby's global variables. Only global variables can be shared between Ruby and Java, when this behavior is chosen. The values of global variables of this type are not kept over the evaluations. <br/>

* Core
```java
ScriptingContainer instance = new ScriptingContainer(LocalVariableBehavior.GLOBAL);
```

* JSR223
```java
System.setProperty("org.jruby.embed.localvariable.behavior", "global");
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("jruby");
```

* BSF
BSF can choose only BSF type.

==== BSF Local Variable Behavior ====

Default for BSF. Local and global variables are available to share between Java and Ruby. Variable names doesn’t start with “$” no matter what the variable types are. Since BSF has a method defined for sharing local variables, it doesn’t confuse. However, core and JSR223 will confuse variables, so don’t use this type for them.<br/>


* Core/JSR223
Don’t choose this behavior.

*BSF
```java
BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
BSFManager manager = new BSFManager();
```
or
```java
System.setProperty("org.jruby.embed.localvariable.behavior", "bsf");
BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
BSFManager manager = new BSFManager();
```

=== Disabling Sharing Variables ===

Scope: Per Evaluation<br/>
Property Name: org.jruby.embed.sharing.variables<br/>
Value: true (default), or false

This option turns on/off a sharing variable feature. Default is on (true). When the feature is off, slight performance improvement will be expected.

* Core
```java
container.setAttribute(AttributeName.SHARING_VARIABLES, false);
```

*JSR223
```java
System.setProperty("org.jruby.embed.sharing.variables", "false");
```


=== CompileMode ===

Scope: Per Container<br/>
Property Name: org.jruby.embed.compilemode<br/>
Value: off (default), jit, or force

JRuby has jit and force options to compile Ruby scripts. This configuration provides users a way of setting a compile mode. When jit or force is specified, only a global variable can be shared between Ruby and Java.

This option is not for pre-compiled Ruby scripts. Ruby2java [http://kenai.com/projects/ruby2java] generated Java classes are not executable on current JRuby Embed.

* Core
```java
ScriptingContainer container = new ScriptingContainer();
container.setCompileMode(CompileMode.JIT);
```


*JSR223/BSF
```java
System.setProperty("org.jruby.embed.compilemode", "jit");
```


=== Ruby Version ===

Scope: Per Container<br/>
Property Name: org.jruby.embed.compat.version<br/>
Value: jruby19, JRuby1_9, ruby1.9….matches j?ruby1[\\._]?9 (case insensitive)

Default Ruby version is 1.8. When you want to use Ruby 1.9 on JRuby Embed, you need to specify the version. In case of Embed Core, the method to set the version is available to use. JSR223 needs system property to recognize the version. BSF judges the registered name. If the system property name or registered name matches the regular expression, [jJ]?(r|R)(u|U)(b|B)(y|Y)1[\\._]?9, Ruby 1.9 is chosen to run scripts. If no system property is there, or matching fails, Ruby 1.8 is chosen.

* Core
```java
ScriptingContainer container = new ScriptingContainer();
// JRuby 1.5.x
container.setCompatVersion(CompatVersion.RUBY1_9);
// JRuby 1.4.0
//container.getProvider().getRubyInstanceConfig().setCompatVersion(CompatVersion.RUBY1_9);
```

* JSR223
```java
System.setProperty("org.jruby.embed.compat.version", "JRuby1.9");
JRubyScriptEngineManager manager = new JRubyScriptEngineManager();
JRubyEngine engine = (JRubyEngine) manager.getEngineByName("jruby");
```

*BSF
```java
BSFManager.registerScriptingEngine("jruby19", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
BSFManager manager = new BSFManager();
manager.exec("jruby19", "ruby/block-param-scope.rb", 0, 0, PathType.CLASSPATH);
```

=== Line Number ===

Scope: Per Evaluation<br/>
Attribute Name: org.jruby.embed.linenumber<br/>
Value: 1, 2, 3,,,, (integer)

Embed Core and BSF have a method argument to set a line number to display for parse errors and backtracks, but JSR223 doesn’t. When you want to specify a line number on JSR223, use the attribute.


* Core
```java
private final String script =
            "puts \"Hello World.\"\n" +
            "puts \"Error is here.";
ScriptingContainer container = new ScriptingContainer();
EvalUnit unit = container.parse(script, 1);
Object ret = unit.run();
```

* JSR223

```java
import javax.script.ScriptContext;
import javax.script.ScriptEngine;
import javax.script.ScriptException;
import org.jruby.embed.jsr223.JRubyScriptEngineManager;

public class LineNumberSample {
    private final String script =
            "puts \"Hello World.\"\n" +
            "puts \"Error is here.";

    private LineNumberSample() throws ScriptException {
        JRubyScriptEngineManager manager = new JRubyScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        try {
            engine.eval(script);    // Since no line number is given, 0 is applied to.
        } catch (Exception e) {
            ;
        }
        try {
            engine.getContext().setAttribute("org.jruby.embed.linenumber", 1, ScriptContext.ENGINE_SCOPE);
            engine.eval(script);
        } catch (Exception e) {
            ;
        }
        try {
            engine.getContext().setAttribute("org.jruby.embed.linenumber", 2, ScriptContext.ENGINE_SCOPE);
            engine.eval(script);
        } catch (Exception e) {
            ;
        }
    }

    public static void main(String[] args)
            throws ScriptException {
        new LineNumberSample();
    }
}
```

Outputs:
```
:1: <script>:2: unterminated string meets end of file (SyntaxError)
:1: <script>:3: unterminated string meets end of file (SyntaxError)
:1: <script>:4: unterminated string meets end of file (SyntaxError)
```

* BSF
```java
private final String script =
            "puts \"Hello World.\"\n" +
            "puts \"Error is here.";
BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
BSFManager manager = new BSFManager();
manager.exec("jruby", “<script>”, 1, 0, script);
```

=== Other Ruby Runtime Configurations ===

If you need more fine-grained control over the Ruby runtime than what's shown here, you can use the various methods of <code>org.jruby.RubyInstanceConfig</code> in Embed Core. Here's the recommended sequence:

```java
ScriptingContainer container = new ScriptingContainer();
RubyInstanceConfig config = container.getProvider().getRubyInstanceConfig();

# First, make the configuration changes you need.
config.someConfigMethod(...);

# Now, you can set variables or run Ruby code.
container.runScriptlet(...);
```

JSR223 and BSF don't support as many configuration options as Embed Core. If you need additional options in these APIs, please file a request in the [http://jra.codehau.org/browe/JRUBY issue tracker].


== Red Bridge Code Examples ==
See the [[RedBridgeExamples]] page.


== Red Bridge Servlet Examples ==
See the [[RedBridgeServletExamples]] page.

== Rewrite it! ==
See the [[RedBridgeWay]] page.

= See Also =
* [[CallingJavafromJRuby|Calling Java from JRuby]]
* [[AccessingJRubyObjectinJava|Accessing JRuby Object in Java]]
* [http://www.blogger.com/posts.g?blogID=22066395&searchType=ALL&txtKeywords=&label=RedBridge JRuby Embed (Red Bridge) info]
* [http://jruby-embed.kenai.com/docs/ JRuby Embed API document]

= Previous Embedding JRuby Page=
We recommend using Embed Core; however, if you're maintaining code that uses the old API, you can find its documentation on the [[JavaIntegration|legacy embedding]] page.
