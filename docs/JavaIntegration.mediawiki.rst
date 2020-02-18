[[Home|&raquo; JRuby Project Wiki Home Page]]
<h1>Embedding JRuby</h1>
__TOC__

==Using the JRuby Interpreter from Java==

Most of the information here has been superceded by the [[RedBridge]] page.  Left here for historic sake, or in case it has clearer examples :)

===Java 6 (using JSR 223: Scripting) ===

Java integration with Java 6 will be using the standard scripting API (JSR223).  A JRuby scripting engine already exists and is located at [https://scripting.dev.java.net/]. 
#Download and unzip the collection of jars from the documents and files section of the site (jsr223-engines.tar.gz or jsr223-engines.zip). 
# Look in the uncompressed files for the <tt>jruby/build/jruby-engine.jar</tt> file.
# Add this file to your classpath and then use the code below to access the engine.<br/> If you're using JRuby 1.1RC3 and 1.1.x on Java 6, use [https://scripting.dev.java.net/servlets/ProjectDocumentList?folderID=8848&expandFolder=8848&folderID=0 version 1.1.2 or later of the JRuby engine].

<pre name="java">
import javax.script.ScriptContext;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
{...}
ScriptEngineManager m = new ScriptEngineManager();
ScriptEngine rubyEngine = m.getEngineByName("jruby");
ScriptContext context = rubyEngine.getContext();

context.setAttribute("label", new Integer(4), ScriptContext.ENGINE_SCOPE);

try{
    rubyEngine.eval("puts 2 + $label", context);
} catch (ScriptException e) {
    e.printStackTrace();
}
</pre>

<tt>ScriptEngine.eval</tt> also takes a <tt>java.io.Reader</tt> object, which allows you to get load scripts from Files or other resource streams very simply, through the same interface.  The context parameter is optional.

If you want to use the scripting API on Java 5, use version 1.1.3 or later of the JRuby engine. Additionally, download <tt>sjp-1_0-fr-ri.zip</tt> from [http://www.jcp.org/en/jsr/detail?id=223], then unzip it and add <tt>script-api.jar</tt> to your classpath. 

Java 5 users might want to use <tt>com.sun.script.jruby.JRubyScriptEngineManager</tt> instead of <tt>javax.script.ScriptEngineManager</tt> to avoid version mismatch errors. However, if you are sure that you don't have any other script engines' archives compiled on JDK 1.6, you can use <tt>javax.script.ScriptEngingManager</tt> to get the engine's instance.

If you can't use Java 6, you can use the Apache Bean Scripting Framework.

When running the compiled code, be sure to use a java invocation similar to the following:

 java -cp .:scripts:bsf.jar:jruby.jar:jruby-engine.jar -Djruby.home=/path/to/jruby/home  my.class.ScriptRunner

The <tt>-Djruby.home</tt> part is necessary so the system can find the ruby libraries.

See [http://wiki.jruby.org/wiki/Walkthroughs_and_Tutorials#JSR_223_scripting Walkthroughs and Tutorials: JSR 223 scripting] for more information.

===Embedding with Bean Scripting Framework (BSF) ===
The [https://github.com/jruby/jruby/wiki/BeanScriptingFramework Bean Scripting Framework], when used with JRuby, will allow you to conveniently to pass your own Java objects to your JRuby script. You can then use these objects in JRuby, and changes will affect your Java program directly. To run a JRuby script using BSF, you must first copy the <tt>BSF.jar</tt> file into your <tt>JAVA_HOME/lib/ext/</tt> folder. Then, try the following:
<pre name="java">
import org.jruby.Ruby.*;
import org.jruby.*;
import org.jruby.javasupport.bsf.*;
import org.apache.bsf.BSFException;
import org.apache.bsf.BSFManager;
{...}
JLabel mylabel = new JLabel();
BSFManager.registerScriptingEngine("ruby", 
                                   "org.jruby.javasupport.bsf.JRubyEngine", 
                                   new String[] { "rb" });

BSFManager manager = new BSFManager();

/* Import an object using declareBean then you can access it in JRuby with $<name> */
 
manager.declareBean("label", mylabel, JFrame.class);
manager.exec("ruby", "(java)", 1, 1, "$label.setText(\"This is a test.\")");
</pre>

===Directly calling JRuby APIs===
See [[DirectJRubyEmbedding]]. The BSF and <tt>javax.scripting</tt> APIs are strongly recommended, as they are most likely to always do the "right thing", which can change over time in the direct version.

==Gotchas==
If you plan on calling gems from an embedded script, there are a couple of things you need to be aware of: 

* If you <tt>require</tt> <tt>'rubygems'</tt>, you need to make sure you set a few system properties: <tt>jruby.base</tt>, <tt>jruby.home</tt>, <tt>jruby.lib</tt>, <tt>jruby.shell</tt>, and <tt>jruby.script</tt>. You can look in <tt>bin/jruby</tt> (a shell script) or <tt>jruby.bat</tt> for examples of setting these properties from the command line. If you can't set them on the command line, you'll need to set them programmatically, or you'll receive a <tt>NullPointerException</tt> when <tt>RbConfigLibrary</tt> loads.

* Make sure you get the load path set properly. Running <tt>jirb</tt> and calling <tt>$LOAD_PATH.inspect</tt> should give you a good idea of what paths need to be included. All those paths can be set in the same way you'd set a Java classpath. However, one reference to <tt>*lib/ruby/1.8*</tt> has to remain relative. This is because some files (<tt>*digest/sha2*</tt>, for example) are loaded from the <tt>jruby.jar</tt>. If you're running unit tests from Ant, you might have problems because Ant tends to expand path elements. Fortunately, it's easy enough to append <tt>lib/ruby/1.8</tt> to the load path before calling <tt>require</tt> <tt>'rubygems'</tt> in your scripts.

Here's an example of a load path from Linux:
<pre>
irb(main):004:0> puts $LOAD_PATH
/home/username/jruby/lib/ruby/site_ruby/1.8
/home/username/jruby/lib/ruby/site_ruby
/home/username/jruby/lib/ruby/1.8
/home/username/jruby/lib/ruby/1.8/java
lib/ruby/1.8
.
=> nil
</pre>

Here's an example of a load path for Windows:
<pre>
C:/common/jruby-0.9.2/lib/ruby/site_ruby/1.8
C:/common/jruby-0.9.2/lib/ruby/site_ruby/1.8/java
C:/common/jruby-0.9.2/lib/ruby/site_ruby
C:/common/jruby-0.9.2/lib/ruby/1.8
C:/common/jruby-0.9.2/lib/ruby/1.8/java
lib/ruby/1.8
</pre>

These settings are specific to my system. Make sure the paths are correct for your system.

If you declare a bean using BSF, make sure you undeclare it when you are done using it, ''even if you declare another bean using the same name''. BSF internally adds declared beans to a vector, and only removes them once they are undeclared.  Or, as an alternative, you can call <tt>registerBean</tt> and access the object from JRuby using the global $bsh reference.

==See also==
* [https://github.com/jruby/jruby/wiki/CallingJavaFromJRuby Calling Java from JRuby]
* [https://github.com/jruby/jruby/wiki/AccessingJRubyObjectInJava Accessing JRuby Object in Java]

==Related Articles==
* [http://leandrosilva.com.br/2008/08/14/executar-jruby-a-partir-do-java Executar JRuby a partir do Java (pt_BR)] Article about integration of Java with JRuby, showing how to run JRuby code from Java.
