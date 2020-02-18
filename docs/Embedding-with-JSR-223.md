Embedding JRuby with JSR223 - Code Examples
===========================================

Simple Evaluation with sharing variables
----------------------------------------

*JSR223
```java
package redbridge;

import javax.script.Bindings;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
import javax.script.SimpleBindings;

public class Jsr223SimpleEvalSample {

    private Jsr223SimpleEvalSample() throws ScriptException {
        System.out.println("[" + getClass().getName() + "]");
        defaultBehavior();
        transientBehavior();
    }

    private void defaultBehavior() throws ScriptException {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        Bindings bindings = new SimpleBindings();
        bindings.put("message", "global variable");
        String script =
                "puts $message";
        engine.eval(script, bindings);
    }

     private void transientBehavior() throws ScriptException {
        System.setProperty("org.jruby.embed.localvariable.behavior", "transient");
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        Bindings bindings = new SimpleBindings();
        bindings.put("message", "local variable");
        bindings.put("@message", "instance variable");
        bindings.put("$message", "global variable");
        bindings.put("MESSAGE", "constant");
        String script =
                "puts message\n" +
                "puts @message\n" +
                "puts $message\n" +
                "puts MESSAGE";
        engine.eval(script, bindings);
    }

    public static void main(String[] args) throws ScriptException {
        new Jsr223SimpleEvalSample();
    }
}
```
Outputs:
```
[redbridge.Jsr223SimpleEvalSample]
global variable
local variable
instance variable
global variable
constant
```

Persistent Local Variables
--------------------------

* JSR223
```java
package redbridge;

import java.util.Set;
import javax.script.Bindings;
import javax.script.ScriptContext;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class Jsr223PersistentLocalVariableSample {

    private Jsr223PersistentLocalVariableSample() throws ScriptException {
        System.out.println("[" + getClass().getName() + "]");
        System.setProperty("org.jruby.embed.localvariable.behavior", "persistent");
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        Object ret = engine.eval("x=144");
        Object ret2 = engine.eval("Math.sqrt x");
        System.out.println("Square root of " + ret + " is " + ret2);

        Bindings bindings = engine.getBindings(ScriptContext.ENGINE_SCOPE);
        String message = "a red big bridge in San Francisco.";
        bindings.put("message", message);
        ret = engine.eval("ret=\"You can see #{message}\"", bindings);
        System.out.println(ret);

        bindings = engine.getBindings(ScriptContext.ENGINE_SCOPE);
        String correction = "elsewhere in the world";
        bindings.put("correction", correction);
        ret = engine.eval("ret = ret.gsub(/in San Francisco/, correction)", bindings);
        System.out.println(ret);

        Set<String> keys = bindings.keySet();
        for (String key : keys) {
            System.out.println(key + ", " + bindings.get(key));
        }
    }

    public static void main(String[] args) throws ScriptException {
        new Jsr223PersistentLocalVariableSample();
    }
}
```
Outputs:
```
[redbridge.Jsr223PersistentLocalVariableSample]
Square root of 144 is 12.0
You can see a red big bridge in San Francisco.
You can see a red big bridge elsewhere in the world.
ret, You can see a red big bridge elsewhere in the world.
MAX_10_EXP, 308
MANT_DIG, 53
DIG, 15
MIN_EXP, -1021
ROUNDS, 1
correction, elsewhere in the world
message, a red big bridge in San Francisco.
MAX, 1.7976931348623157E308
RADIX, 2
EPSILON, 2.220446049250313E-16
MIN, 4.9E-324
MIN_10_EXP, -307
x, 144
MAX_EXP, 1024
```

Method Call
-----------

* JSR223
```java
package redbridge;

import javax.script.Invocable;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class Jsr223MethodCallSample {

    private Jsr223MethodCallSample() throws ScriptException, NoSuchMethodException {
        System.out.println("[" + getClass().getName() + "]");
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        String script =
                "# Radioactive decay\n" +
                "def amount_after_years(q0, t)\n" +
                  "q0 * Math.exp(1.0 / $half_life * Math.log(1.0/2.0) * t)\n" +
                "end\n" +
                "def years_to_amount(q0, q)\n" +
                  "$half_life * (Math.log(q) - Math.log(q0)) / Math.log(1.0/2.0)\n" +
                "end";
        Object receiver = engine.eval(script);
        
        engine.put("half_life", 5715); // Carbon
        String method = "amount_after_years"; // calculates the amount left after given years
        Object[] args = new Object[2];
        args[0] = 10.0;    // initial amount is 10.0g
        args[1] = 1000;    // suppose 1000 years have passed
        Object result = ((Invocable)engine).invokeFunction(method, args);
        System.out.println(args[0] + "g Carbon to decay to " + result + "g in " + args[1] + " years");

        method = "years_to_amount"; // calculates the years to decay to a given amount
        args[0] = 10.0;    // initial amount is 10.0g
        args[1] = 1.0;     // suppose 1.0g is still there
        result = ((Invocable)engine).invokeFunction(method, args);
        System.out.println(args[0] + "g Carbon to decay to " + args[1] + "g in " + result + " years");

    }

    public static void main(String[] args) throws ScriptException, NoSuchMethodException {
        new Jsr223MethodCallSample();
    }
}
```
Outputs:
```
[redbridge.Jsr223MethodCallSample]
10.0g Carbon to decay to 8.857809480593293g in 1000 years
10.0g Carbon to decay to 1.0g in 18984.81906228128 years
```

Class Method Call
-----------------

* tree_with_ivars.rb
```rb
class Tree
  attr_reader :name, :shape, :foliage, :flower
  def initialize(flower)
    @flower = flower
  end
  def to_s
    "#{name.capitalize} is a #{shape} shaped, #{foliage} tree, and blooms #{flower.color} flowers in #{flower.bloomtime}."
  end
  def update
    flower.color = @color
    flower.bloomtime = @bloomtime
  end
end

class Flower
  attr_accessor :color, :bloomtime
  def initialize
  end
end

Tree.new(Flower.new)
```

* JSR223
```java
package redbridge;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.Reader;
import javax.script.Invocable;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class Jsr223ClassMethodCallSample {
    private final static String filename = "ruby/tree_with_ivars.rb";

    private Jsr223ClassMethodCallSample() throws ScriptException, NoSuchMethodException, FileNotFoundException {
        System.out.println("[" + getClass().getName() + "]");
        System.setProperty("org.jruby.embed.localvariable.behavior", "transient");
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");

        String basedir = System.getProperty("user.dir") + "/src";
        Reader reader = new FileReader(basedir + "/" + filename);
        Object receiver = engine.eval(reader);
        engine.put("@name", "cherry blossom");
        engine.put("@shape", "oval");
        engine.put("@foliage", "deciduous");
        engine.put("@color", "pink");
        engine.put("@bloomtime", "March - April");
        ((Invocable)engine).invokeMethod(receiver, "update");
        System.out.println(((Invocable)engine).invokeMethod(receiver, "to_s"));

        engine.put("@name", "cedar");
        engine.put("@shape", "pyramidal");
        engine.put("@foliage", "evergreen");
        engine.put("@color", "nondescript");
        engine.put("@bloomtime", "April - May");
        ((Invocable)engine).invokeMethod(receiver, "update");
        System.out.println(((Invocable)engine).invokeMethod(receiver, "to_s"));
    }

    public static void main(String[] args) throws ScriptException, NoSuchMethodException, FileNotFoundException {
        new Jsr223ClassMethodCallSample();
    }
}
```
Outputs:
```
[redbridge.Jsr223ClassMethodCallSample]
Cherry blossom is a oval shaped, deciduous tree, and blooms pink flowers in March - April.
Cedar is a pyramidal shaped, evergreen tree, and blooms nondescript flowers in April - May.
```

Interface Implementation
------------------------

** Interface: vanilla.Calculable
```java
package vanilla;

import java.util.List;

public interface Calculable {
    List<Long> dimension(long base);
    Double hypotenuse(double adjacent, double opposite);
}
```

** Implementation: calculation.rb
```rb
class Calculation
  include Java::vanilla.Calculable
  def dimension(base)
    x = base + 1
    y = base + 2
    z = base - 1
    return x, y, z
  end
  def hypotenuse(adjacent, opposite)
    Math.hypot(adjacent, opposite)
  end
end
Calculation.new
```
** Interface: vanilla.FluidForce
```java
package vanilla;

public interface FluidForce {
    Double getFluidForce(double a, double b, double depth);
}
```
** Implementation: fluid_force.rb
```rb
def get_fluid_force(x, y, depth)
  area = Math::PI * x * y # ellipse
  return @w * area * depth
end
```

* JSR223
** Java program
```java
package redbridge;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.util.List;
import javax.script.Bindings;
import javax.script.Invocable;
import javax.script.ScriptContext;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
import vanilla.Calculable;
import vanilla.FluidForce;

public class Jsr223GetInterfaceSample {

    private Jsr223GetInterfaceSample() throws ScriptException, FileNotFoundException {
        System.out.println("[" + getClass().getName() + "]");
        System.setProperty("org.jruby.embed.localvariable.behavior", "transient");
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");

        // implemented by a Ruby class
        String filename = System.getProperty("user.dir") + "/src/ruby/calculation.rb";
        FileReader reader = new FileReader(filename);
        Object receiver = engine.eval(reader);
        Calculable c = (Calculable)((Invocable)engine).getInterface(receiver, Calculable.class);
        List<Long> xyz = c.dimension(20L);
        System.out.format("Dimensions are %d x %d x %d.\n", xyz.get(0), xyz.get(1), xyz.get(2));
        double adjacent = 5.0;
        double opposite = 12.0;
        Double hypotenuse = c.hypotenuse(adjacent, opposite);
        System.out.format("Adjacent, opposite, and hypotenuse are %.2f, %.2f, %.2f.\n",
                adjacent, opposite, hypotenuse);

        // implemented by a top level function
        filename = System.getProperty("user.dir") + "/src/ruby/fluid_force.rb";
        reader = new FileReader(filename);
        Bindings bindings = engine.getBindings(ScriptContext.ENGINE_SCOPE);
        bindings.put("@w", 49.4); // weight-densities of ethyl alcohol in pounds per cubic foot
        engine.eval(reader, bindings);
        FluidForce f = (FluidForce)((Invocable)engine).getInterface(FluidForce.class);
        double a = 2.0;
        double b = 3.0;
        double depth = 6.0;
        Double force = f.getFluidForce(a, b, depth);
        System.out.format("Ethyl alcohol force to %.2f ft x %.2f ft ellipse in depth of %.2f ft is %.5f lb.\n",
                a, b, depth, force);
    }

    public static void main(String[] args) throws ScriptException, FileNotFoundException {
        new Jsr223GetInterfaceSample();
    }
}
```
Outputs:
```
[redbridge.Jsr223GetInterfaceSample]
Dimensions are 21 x 22 x 19.
Adjacent, opposite, and hypotenuse are 5.00, 12.00, 13.00.
Ethyl alcohol force to 2.00 ft x 3.00 ft ellipse in depth of 6.00 ft is 5587.00838 lb.
```

Parse Once, Eval Many Times
---------------------------

** count_down.rb
```rb
require 'date'

def count_down_birthday
  now = DateTime.now
  year = now.year
  days = DateTime.new(year, @month, @day).yday - now.yday
  if days < 0
    this_year = DateTime.new(year, 12, 31).yday - now.yday
    next_year = DateTime.new(year + 1, @month, @day).yday
    days = this_year + next_year
  end
  return "Happy Birthday!" if days == 0
  return "You have #{days} day(s) to your next birthday!"
end
count_down_birthday
```

* JSR223
** Java program
```java
package redbridge;

import java.io.FileNotFoundException;
import java.io.FileReader;
import javax.script.Compilable;
import javax.script.CompiledScript;
import javax.script.ScriptContext;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class Jsr223CompiledScriptSample {
    private static final String filename = "src/ruby/count_down.rb";

    private Jsr223CompiledScriptSample() throws ScriptException, FileNotFoundException {
        System.out.println("[" + getClass().getName() + "]");
        System.setProperty("org.jruby.embed.localvariable.behavior", "transient");
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("jruby");
        String scriptFile = System.getProperty("user.dir") + "/" + filename;
        FileReader reader = new FileReader(scriptFile);
        CompiledScript cs = ((Compilable)engine).compile(reader);
        evaluate(engine, cs, 10, 11);
        evaluate(engine, cs, 1, 1);
        evaluate(engine, cs, 12, 31);
        evaluate(engine, cs, 7, 4);
    }

    private void evaluate(ScriptEngine engine, CompiledScript cs, int month, int day)
            throws ScriptException {
        engine.put("@month", month);
        engine.put("@day", day);
        Object ret = cs.eval();
        System.out.println(ret);
        engine.getBindings(ScriptContext.ENGINE_SCOPE).clear();
    }

    public static void main(String[] args) throws ScriptException, FileNotFoundException {
        new Jsr223CompiledScriptSample();
    }
}
```
Outputs:
```
[redbridge.Jsr223CompiledScriptSample]
Happy Birthday!
You have 82 day(s) to your next birthday!
You have 81 day(s) to your next birthday!
You have 266 day(s) to your next birthday!
```

Servlet Examples
----------------

See [[RedBridgeServletExamples]] page.