__TOC__


==Direct Embedding==

JRuby can also be embedded directly using JRuby's own APIs. The interfaces in JavaEmbedUtils are meant to be long-lasting API's for raw embedding (that slightly preceded the RedBridge embed core ones). This is now generally only recommended for people embedding into other scripting engines and frameworks or for those who need a "bare metal" approach and are willing to keep up with continuing JRuby API changes. It is strongly recommended for most embedders to use BSF or javax.scripting APIs to embed JRuby. See [[RedBridge]].

===Instantiating the JRuby interpreter===
The most direct method of running JRuby in Java is as follows (works for JRuby 1.1+):

```java
import java.util.*;
import org.jruby.Ruby;
import org.jruby.RubyRuntimeAdapter;
import org.jruby.javasupport.JavaEmbedUtils;  
{...}
// Create runtime instance
List<String> loadPaths = new ArrayList<String>();
Ruby runtime = JavaEmbedUtils.initialize(loadPaths);
RubyRuntimeAdapter evaler = JavaEmbedUtils.newRuntimeAdapter();

{...}
evaler.eval(runtime, "puts 1+2");
{...}

// Shutdown and terminate instance
JavaEmbedUtils.terminate(runtime);
```

You could also instantiate it using even deeper "raw" API's--check the source to [[Main.java|https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/Main.java]] and [[ScriptingContainer.java|https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/embed/ScriptingContainer.java]] to see how they do it (RubyConfigAdapter et al), but it's pretty complex, and using "some other way" (embedutils, or ScriptingContainer RedBridge) will get you a runtime running more simply.

===Raw Example===

In mirah, here's a somewhat low-level example. See below for others that use "eval" or the like.

```Mirah
import org.jruby.*
import org.jruby.javasupport.JavaEmbedUtils
import java.util.Collections

runtime = JavaEmbedUtils.initialize(Collections.emptyList())
longy = RubyFixnum.new(runtime, 35) # type RubyFixnum
puts longy # 35
yo = longy.op_plus(runtime.getCurrentContext, RubyFixnum.new(runtime, 35)) # type RubyIObject since op_plus returns one of those.  The result value happens to be another RubyFixnum, which descends from RubyIObject
puts yo # 70
...
```

===Subclassing a Java class===

Here is another raw example, using eval et al.

```java
/*
This is run using
java -cp .:/path/to/jruby.jar  RubyFromJava

*/

import org.jruby.*;
import org.jruby.javasupport.JavaEmbedUtils;  
import java.util.ArrayList;

public class RubyFromJava {
	public String fooString() {
		return "foo";
	}
	
	public static void main(String[] ARGV) {
		System.out.println("Started");
		RubyFromJava sTest = new RubyFromJava();
		
		System.out.println("boring: " + sTest.fooString() + "\n"); // outputs foo
		
		/* This script subclasses the java Object class.
		It is then instantiated and the foobarString method is called. */
		
		String script = 
			"require 'java'\n" +
			"class RSubclass1 < java.lang.Object\n" + // subclassing java.* is magic
			"	def foobarString\n" +
			"		return @returnString = toString() + 'BAR'\n" +
			"	end\n" +
			"end\n" +
			"rsubclass = RSubclass1.new\n" +
			"puts rsubclass.foobarString\n";
    
                Ruby runtime = JavaEmbedUtils.initialize(new ArrayList());
                RubyRuntimeAdapter evaler = JavaEmbedUtils.newRuntimeAdapter();

    		evaler.eval(runtime, script); // outputs org.jruby.proxy.java.lang.Object$Proxy0@1c7f37dBAR

		System.out.println("-----------------\n");

		/* This script subclasses the RubyFromJava class.
		It is then instantiated and the foobarString method is called. */

		script = 
			"require 'java'\n" +
                        "class RSubclass2 < Java::RubyFromJava\n" + // subclassing non {org,com}.* classpath requires you prefix the class with Java:: or java.
			"	def foobarString\n" +
			"		return @returnString = fooString() + 'BAR'\n" +
			"	end\n" +
			"\n" +
			"rsubclass = RSubclass2.new\n" +
			"puts(rsubclass.foobarString())\n" +
			"end";
                 evaler.eval(runtime, script); // outputs fooBAR
	}
}
```

===Using a Ruby subclass by casting it to its Java superclass -- from Java!===

```java
/*
This is run using
java -cp .:/path/to/jruby.jar  RubyFromJava

*/


import org.jruby.*;
import org.jruby.javasupport.JavaEmbedUtils;  
import java.util.ArrayList;

public class RubyFromJava {
	static RubyFromJava globalRFJ;
	
	public String fooString() {
		return "foo";
	}
	
	public static void main(String[] ARGV) {
		System.out.println("Started");
		RubyFromJava sTest = new RubyFromJava();
		
		System.out.println("boring: " + sTest.fooString() + "\n");
		
		String script = 
			"require 'java'\n" +
			"class RSubclass < Java::RubyFromJava\n" + // subclassing non java.* requires you prefix the class with Java::
			"	def fooString\n" +
			"		return super + 'BAR!'\n" +
			"	end\n" +
			"end";
      
                Ruby runtime = JavaEmbedUtils.initialize(new ArrayList());
                RubyRuntimeAdapter evaler = JavaEmbedUtils.newRuntimeAdapter();

		evaler.eval(runtime, script);
      
		Object rfj = evaler.eval(runtime, "RSubclass.new()");
		rfj = org.jruby.javasupport.JavaEmbedUtils.rubyToJava(runtime, (org.jruby.runtime.builtin.IRubyObject) rfj, RubyFromJava.class);
                // rubyToJava basically tries to convert a ruby object to its "raw" native java object.  If it's a class then it will converts its (ruby instance) wrapped in a dynamic proxy handler that intercepts calls and directs them to the ruby class (if implemented) or to the internal java class which is actually stored as an internal element.  But the details don't matter as much.
		System.out.println("Local: " + ((RubyFromJava) rfj).fooString() + "\n"); // outputs Local: fooBAR!
	}
}
```

----

'''Summary - how to do this''' <br/>
1. Write a Java class X (here in example RubyFromJava.java) which should be subclassed into a Ruby class Y. Let be fooString() be a method which you would like to overwrite in Ruby. This method will be called directly from Java, yet execute Ruby code.

2. Get Ruby runtime instance via
```java
  Ruby runtime = Ruby.getDefaultInstance();
  // 
  Ruby runtime = JavaEmbedUtils.initialize(new ArrayList());
```

3. Parse your script with Ruby class Y which overwrites fooString(). Assuming that the script is contained in the String "script", you do this via
```java
  RubyRuntimeAdapter evaler = JavaEmbedUtils.newRuntimeAdapter();
  evaler.eval(runtime, script);
  // or IRubyObject rubyObject = evaler.parse(runtime, script_string, filename, line).run());
```

4. Intantiate Ruby class Y and cast (?) it to Java class X via
```java
  IRubyObject y = evaler.eval(runtime, "Y.new()");
  X y2 = org.jruby.javasupport.JavaEmbedUtils.rubyToJava(runtime, y, X.class);
```

5. Now call the method fooString() of the object y - Ruby code is executed!
```java
  y2.fooString();
```

'''Remark 1''': 

The overwritten method can also take parameters (at least Java objects, not sure about Ruby objects). So fooString() can be changed into:
```java
  public String fooString(String myArg) {
      return "foo " + myArg;
  }
```
and the Ruby overwriting method then might look like this:
```ruby
  def fooString (myArg)
    return 'BAR!' + myArg
  end
```
You call then the method as:
```java
  y2.fooString("Argument");
```

'''Remark 2''': 

Be aware that the Ruby object y (in the code above) has been created from "scratch" (because neither it nor its parent have an initializer) so its fields haven't been initialized.  You can define a constructor in java-land, or ruby-land.  Eventually, a java-land constructor must and will be invoked.  If you define a ruby-land constructor, you can either call super within your constructor (at any time--if you pass any number of parameters they will be passed on to the right constructor, if you pass no parameters it will pass on the same parameters your initialize method received, if you pass on a different number of parameters then it will call the parent constructor that takes that many parameters). If you never call super within your initialize method, or it will be called implicitly *after* your constructor finishes, with the same parameters your constructor received.  

=== Passing Java parameters to a Ruby object's new method ===

```java
IRubyObject rubyClass = evaler.eval(runtime, "MyRubyClass");
Object[] parameters = {javaObject, otherJavaObject};
JavaEmbedUtils.invokeMethod(runtime, rubyClass, "new", parameters, IRubyObject.class);
```

If your Ruby subclass is extending or implementing a Java type, you can set the return type parameter and cast the return value of invokeMethod to the appropriate type.

```java
IRubyObject rubyClass = evaler.eval(runtime, "MyRubyClass");
Object[] parameters = {javaObject, otherJavaObject};
MyJavaType rubyObject = (MyJavaType)JavaEmbedUtils.invokeMethod(runtime, rubyClass "new", parameters, MyJavaType.class);
```


===Ruby code in your classpath, Mixed with java===
For those who don't want BSF, and want to have lots of their application code in ruby files on their classpath, the following very simple utility can be useful (feel free to re-use as is):

```java
package ruby.utils;

import java.util.ArrayList;

import org.jruby.Ruby;
import org.jruby.RubyRuntimeAdapter;
import org.jruby.javasupport.JavaEmbedUtils;
import org.jruby.runtime.builtin.IRubyObject;

/**
 * This utility is a simple way to keep most of your code in ruby, 
 * and must pass across a "root" object from java into a "root" object
 * on the ruby side (calling a single argument method you specify - the root ruby object is created for you).
 * Ruby code can live on the the classpath, next to your java.
 * This doesn't require BSF, or any mandatory dependencies other then jruby.jar.
 * 
 * @author <a href="mailto:michael.neale@gmail.com">Michael Neale</a> 
 */
public class RubyLauncher {

    /** this is the root object - to be used over and over */
    private IRubyObject rootRubyObject;
    private Ruby runtime;

    /**
     * 
     * @param initialRequire The name of the .rb file that is your starting point (on your claspath).
     * @param rootRubyClass The name of the ruby class in the above .rb file, must have no-arg constructor (a new instance will be created).
     * @param rootMethod The name of the method to call in the above class when "call" is called.
     */
    public RubyLauncher(String initialRequire, String rootRubyClass, String rootMethod) {
        
        String bootstrap = 
            "require \"" + initialRequire +  "\"\n"+
            "class Bootstrap \n" +
            "   def execute root_object  \n" +          
            "       " + rootRubyClass + ".new." + rootMethod + "(root_object) \n" +
            "   end    \n" +      
            "end \n" +
            "Bootstrap.new";

        // This list holds the directories where the Ruby scripts can be found; unless you have complete 
        // control how jruby is launched, use absolute paths
        List<String> loadPaths = new ArrayList<String>();
	loadPaths.add(".");
        
        runtime = JavaEmbedUtils.initialize( loadPaths );
        rootRubyObject = JavaEmbedUtils.newRuntimeAdapter().eval( runtime, bootstrap );
    }
    
    /**
     * This can be called over and over on the one instance.
     * 
     * Pass your root java object to the root ruby object (which was created in the constructor, with the specified method). 
     * If you want to get data out, best bet is to make the root object(s) wrappers for in/out objects.
     */
    public void call(Object obj) {
        JavaEmbedUtils.invokeMethod( runtime, rootRubyObject, "execute", new Object[] {obj}, null );
    }
    
    /**
      * Use this method when embedding ruby files within a jar. They won't be found on the classpath or LOAD_PATH
      * unless added relative to the location of the jar.
      *
      * e.g. jar structure:  /com/example/rubyfiles/my_class.rb
      * loadPaths.add(getPathToJar("/com/example/rubyfiles/");
      *
      * @param jar_internal_path Absolute path to your ruby files inside the jar file
      * @return String Path URL added to JRuby's LOAD_PATH
      */
    private String getPathToJar(String jar_internal_path)
    {
        java.net.URL url =  RubyLauncher.class.getResource(jar_internal_path);
        return url.getPath();
    }
}
```

To use this is simple, you create a "root" ruby object in a .rb file on your classpath, and pass the name/path of that file into the constructor (as well as the initial method to pass the root java object into).
Note that you can require code that lives elsewhere on your classpath, just as if it is on the filesystem.

For example:
```java
  Map<String, Object> root = new HashMap<String, Object>();
  root.put("name", "david");

  RubyLauncher launcher = new RubyLauncher("c:/ruby/myscripts/ruby_root.rb", "RubyRoot", "start");
  launcher.call(root);
```

NOTE: I was unable to get the RubyLauncher to find the Ruby script on my classpath. So I hardcoded the path.

"RubyRoot" is the name of a ruby class in ruby_root.rb, and "start" is the name of a method that will take the "nae" object when "call" is invoked. Note that the initial Ruby script can <i>require</i> other .rb files from the classpath (just use the path, like you would on the filesystem) relative to where the "ruby_root.rb" file is.

```ruby
class RubyRoot
  
  def start( root )
    puts root.toString
  end
  
end
```

Easy ! And no extra dependencies. This way you can just have minimal java bootstrap code, fire up the RubyLauncher once, and away you go.
