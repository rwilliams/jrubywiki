Two Examples of accessing Ruby classes in Java use a script Engine.  The first example uses Java 5 and previous versions.  The second example uses Java 6 and the the scripting engine.

Java 5 Example
==============

This example shows your can access the JRuby Object in Java 5 , it is not as elegant as the solution in Java 6 but that discussion will follow later.  Specifically this example uses `getDefaultInstance' to instantiate a Ruby environment.

We then added Java variables to the script String and evaled that string.  The Java variables were "yamled" to  the specific file.  To do this correctly, there are certain dependencies that must added to the build path.  The Jruby complete jar for Ruby stdlib.

````java
package jrubyfromJava;

import org.jruby.*;

public class JrubyTest { 
    public static void main(String[] ARGV) {
        System.out.println("starting"); 
        Ruby runtime = Ruby.getDefaultInstance(); 
        System.out.println("ruby instantiated"); 
        String script = "require 'java'\n"+ 
                        "require 'yaml'\n"+ 
                        "testObj = Object.new\n"+ 
                        "puts 'running java'\n"+ 
                        //"File.open('test.yml','w') { |out| YAML.dump(testInfoObj, out )}\n" + 
                        "puts 'closed file'\n" + 
                        " testObj  = File.open('test.yml', 'r'){|yf| YAML.load(yf)}\n" + 
                        "puts testObj";

        runtime.evalScript(script); 
        //script engine test.   
    }
}
````

Java 6
======

Java 6 handles the accessing of Java variables more handily. When the engine gets instantiated a context for each variable that you want to pass in and out of the eval string is set.  Now we can access the variable after it has been evaled by JRuby. The changes are retained within the set Context.

````java
package jrubyfromJava;

import javax.script.ScriptContext; 
import javax.script.ScriptEngine; 
import javax.script.ScriptEngineManager; 
import javax.script.ScriptException; 

public class scriptEngineTest { 
    /** 
     * @param args 
     */ 
    public static void main(String[] args) { 
        ScriptEngineManager m = new ScriptEngineManager(); 
        ScriptEngine rubyEngine = m.getEngineByName("jruby"); 
        //setting the Context for the label variable
        rubyEngine.getContext().setAttribute("label",new Float(1.0), ScriptContext.ENGINE_SCOPE); 
        
        //ScriptContext context = null; 
        try { 
            rubyEngine.eval("require 'java'\n"+ 
                            "require 'yaml'\n"+ 
                            "testObj = Object.new\n"+ 
                            "puts 'running java'\n"+ 
                            //"File.open('test.yml','w') { |out| YAML.dump(testInfoObj, out )}\n" + 
                            "puts 'closed file'\n" + 
                            "f = File.open('test.yml', 'r'){|yf| YAML.load(yf)}\n" + 
                            "puts testObj\n" +
                            "$nv = f.name+f.version.to_s"); 
                            
            Object nv = rubyEngine.getContext().getAttribute("nv"); 
            System.out.println(nv.toString()); 
        } catch (ScriptException e) { 
            // TODO Auto-generated catch block 
            e.printStackTrace(); 
        } 
    }
}
````

The Object.new is not the object code I used.  I had to change it because it was private.  If you change it to array it will work just fine.
