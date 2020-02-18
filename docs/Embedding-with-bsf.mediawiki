<h1>Embedding JRuby with BSF - Code Examples</h1>

__TOC__

== Code Examples ==

=== Simple Evaluation with sharing variables ===

*BSF
```java
package azuki;

import org.apache.bsf.BSFException;
import org.apache.bsf.BSFManager;

public class BsfSimpleEvalSample {
    private BsfSimpleEvalSample() throws BSFException {
        System.out.println("[" + getClass().getName() + "]");
        BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
        BSFManager manager = new BSFManager();
        manager.declareBean("message", "global variable", String.class);
        String script =
                "puts $message";
        manager.exec("jruby", "<script>", 0, 0, script);
    }

    public static void main(String[] args) throws BSFException {
        new BsfSimpleEvalSample();
    }
}
```
Outputs:
```
[azuki.BsfSimpleEvalSample]
global variable
```

=== Persistent Local Variables ===

* BSF
```java
package azuki;

import java.util.Vector;
import org.apache.bsf.BSFException;
import org.apache.bsf.BSFManager;

public class BsfPersistentLocalVariableSample {
    private BsfPersistentLocalVariableSample() throws BSFException {
        System.out.println("[" + getClass().getName() + "]");
        BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
        BSFManager manager = new BSFManager();

        Object ret = manager.apply("jruby", "<script>", 0, 0, "x=144", null, null);
        Object ret2 = manager.apply("jruby", "<script>", 0, 0, "Math.sqrt x", null, null);
        System.out.println("Square root of " + ret + " is " + ret2);

        Vector paramNames = new Vector();
        Vector args = new Vector();
        paramNames.add("message");
        args.add("red small beans and often used in a form of paste.");
        ret = manager.apply("jruby", "<script>", 0, 0, "ret=\"Azuki beans are #{message}\"", paramNames, args);
        System.out.println(ret);
        paramNames.clear();
        args.clear();
        
        paramNames.add("correction");
        args.add("usually");
        ret = manager.apply("jruby", "<script>", 0, 0, "ret = ret.gsub(/often/, correction)", paramNames, args);
        System.out.println(ret);
    }

    public static void main(String[] args) throws BSFException {
        new BsfPersistentLocalVariableSample();
    }
}
```
Outputs:
```
[azuki.BsfPersistentLocalVariableSample]
Square root of 144 is 12.0
Azuki beans are red small beans and often used in a form of paste.
Azuki beans are red small beans and usually used in a form of paste.
```

=== Method Call ===

* BSF
```java
package azuki;

import org.apache.bsf.BSFException;
import org.apache.bsf.BSFManager;
import org.jruby.embed.bsf.JRubyEngine;

public class BsfMethodCallSample {
    private BsfMethodCallSample() throws BSFException {
        System.out.println("[" + getClass().getName() + "]");
        BSFManager.registerScriptingEngine("jruby", "org.jruby.embed.bsf.JRubyEngine", new String[] {"rb"});
        BSFManager manager = new BSFManager();
        JRubyEngine engine = (JRubyEngine) manager.loadScriptingEngine("jruby");
        String script =
                "# Radioactive decay\n" +
                "def amount_after_years(q0, t)\n" +
                  "q0 * Math.exp(1.0 / $half_life * Math.log(1.0/2.0) * t)\n" +
                "end\n" +
                "def years_to_amount(q0, q)\n" +
                  "$half_life * (Math.log(q) - Math.log(q0)) / Math.log(1.0/2.0)\n" +
                "end";
        Object receiver = manager.eval("jruby", "radioactive_decay", 0, 0, script);

        String method = "amount_after_years"; // calculates the amount left after given years
        Object[] args = new Object[2];
        args[0] = 10.0;    // initial amount is 10.0g
        args[1] = 1000;    // suppose 1000 years have passed

        // Radium
        manager.declareBean("half_life", 1599, Long.class); // the half-life of Radium is 1599
        Object result = engine.call(receiver, method, args);
        System.out.println(args[0] + "g Radium to decay to " + result + "g in " + args[1] + " years");
        
        method = "years_to_amount"; // calculates the years to decay to a given amount
        args[0] = 10.0;    // initial amount is 10.0g
        args[1] = 1.0;     // suppose 1.0g is still there
        result = engine.call(receiver, method, args);
        System.out.println(args[0] + "g Radium to decay to " + args[1] + "g in " + result + " years");
    }

    public static void main(String[] args) throws BSFException {
        new BsfMethodCallSample();
    }

}
```
Outputs:
```
[azuki.BsfMethodCallSample]
10.0g Radium to decay to 6.482441247843886g in 1000 years
10.0g Radium to decay to 1.0g in 5311.763023724893 years
```

== Servlet Examples ==
See [[RedBridgeServletExamples]] page.