[[Home|&raquo; JRuby Project Wiki Home Page]]<br/>
[[RedBridge|&raquo; Embedding JRuby Wiki Page]]
<h1>Embedding JRuby - Code Examples</h1>

If you are looking for JSR223 (javax.scripting) or BSF examples, visit the following pages:

[[Embedding-with-JSR-223|Embedding with JSR-223]]

[[Embedding-with-bsf|Embedding with BSF]]

__TOC__
= Red Bridge =

== Features of Red Bridge ==
See [[RedBridge#Features_of_Red_Bridge|Features of Red Bridge]] section.

== Download ==
See [[RedBridge#Download|Download]] section.

== Getting Started ==
See [[RedBridge#Getting_Started|Getting Started]] section.

== Configurations ==
See [[RedBridge#Configurations|Configurations]] section.

== Code Examples ==

=== Simple Evaluation with sharing variables ===

* Core
```java
package vanilla;

import org.jruby.embed.ScriptingContainer;

public class SimpleEvalSample {

    private SimpleEvalSample() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer();
        container.put("message", "local variable");
        container.put("@message", "instance variable");
        container.put("$message", "global variable");
        container.put("MESSAGE", "constant");
        String script =
                "puts message\n" +
                "puts @message\n" +
                "puts $message\n" +
                "puts MESSAGE";
        container.runScriptlet(script);
    }

    public static void main(String[] args) {
        new SimpleEvalSample();
    }
}
```
Outputs:
```
[vanilla.SimpleEvalSample]
local variable
instance variable
global variable
constant
```

=== Persistent Local Variables ===

* Core
```java
package vanilla;

import java.util.Map;
import java.util.Set;
import org.jruby.embed.ScriptingContainer;
import org.jruby.embed.LocalVariableBehavior;

public class PersistentLocalVariableSample {

    private PersistentLocalVariableSample() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer(LocalVariableBehavior.PERSISTENT);
        Object ret = container.runScriptlet("x=144");
        Object ret2 = container.runScriptlet("Math.sqrt x");
        System.out.println("Square root of " + ret + " is " + ret2);

        String message = "hot Vanilla Latte at that cafe.";
        container.put("message", message);
        ret = container.runScriptlet("ret=\"You can enjoy #{message}\"");
        System.out.println(ret);

        String correction = "could have enjoyed";
        container.put("correction", correction);
        ret = container.runScriptlet("ret = ret.gsub(/can enjoy/, correction)");
        System.out.println(ret);
        
        Map m = container.getVarMap();
        Set<String> keys = container.getVarMap().keySet();
        for (String key : keys) {
            System.out.println(key + ", " + m.get(key));
        }
    }

    public static void main(String[] args) {
        new PersistentLocalVariableSample();
    }
}
```
Outputs:
```
[vanilla.PersistentLocalVariableSample]
Square root of 144 is 12.0
You can enjoy hot Vanilla Latte at that cafe.
You could have enjoyed hot Vanilla Latte at that cafe.
ret, You could have enjoyed hot Vanilla Latte at that cafe.
MANT_DIG, 53
MAX_10_EXP, 308
DIG, 15
MIN_EXP, -1021
ROUNDS, 1
correction, could have enjoyed
message, hot Vanilla Latte at that cafe.
MAX, 1.7976931348623157E308
RADIX, 2
EPSILON, 2.220446049250313E-16
MIN, 4.9E-324
MIN_10_EXP, -307
x, 144
MAX_EXP, 1024
```

=== Method Call ===

* Core
```java
package vanilla;

import org.jruby.embed.ScriptingContainer;

public class MethodCallSample {

    private MethodCallSample() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer();
        String script =
                "# Radioactive decay\n" +
                "def amount_after_years(q0, t)\n" +
                  "q0 * Math.exp(1.0 / $half_life * Math.log(1.0/2.0) * t)\n" +
                "end\n" +
                "def years_to_amount(q0, q)\n" +
                  "$half_life * (Math.log(q) - Math.log(q0)) / Math.log(1.0/2.0)\n" +
                "end";
        Object receiver = container.runScriptlet(script);

        container.put("$half_life", 24100); // Plutonium
        String method = "amount_after_years"; // calculates the amount left after given years
        Object[] args = new Object[2];
        args[0] = 10.0;    // initial amount is 10.0g
        args[1] = 1000;    // suppose 1000 years have passed
        Object result = container.callMethod(receiver, method, args, Double.class);
        System.out.println(args[0] + "g Plutonium to decay to " + result + "g in " + args[1] + " years");

        method = "years_to_amount"; // calculates the years to decay to a given amount
        args[0] = 10.0;    // initial amount is 10.0g
        args[1] = 1.0;     // suppose 1.0g is still there
        result = container.callMethod(receiver, method, args, Double.class);
        System.out.println(args[0] + "g Plutonium to decay to " + args[1] + "g in " + result + " years");
    }

    public static void main(String[] args) {
        new MethodCallSample();
    }
}
```
Outputs:
```
[vanilla.MethodCallSample]
10.0g Plutonium to decay to 9.716483752784367g in 1000 years
10.0g Plutonium to decay to 1.0g in 80058.46708678544 years
```

=== Class Method Call ===
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

* Core
```java
package vanilla;

import org.jruby.embed.PathType;
import org.jruby.embed.ScriptingContainer;

public class ClassMethodCallSample {
    private final static String filename = "ruby/tree_with_ivars.rb";

    private ClassMethodCallSample() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer();
        
        Object receiver = container.runScriptlet(PathType.CLASSPATH, filename);
        container.put(receiver, "@name", "cherry blossom");
        container.put(receiver, "@shape", "oval");
        container.put(receiver, "@foliage", "deciduous");
        container.put(receiver, "@color", "pink");
        container.put(receiver, "@bloomtime", "March - April");
        container.callMethod(receiver, "update", Object.class);
        System.out.println(container.callMethod(receiver, "to_s", String.class));

        container.put(receiver, "@name", "cedar");
        container.put(receiver, "@shape", "pyramidal");
        container.put(receiver, "@foliage", "evergreen");
        container.put(receiver, "@color", "nondescript");
        container.put(receiver, "@bloomtime", "April - May");
        container.callMethod(receiver, "update", Object.class);
        System.out.println(container.callMethod(receiver, "to_s", String.class));
    }

    public static void main(String[] args) {
        new ClassMethodCallSample();
    }
}
```
Outputs:
```
[vanilla.ClassMethodCallSample]
Cherry blossom is a oval shaped, deciduous tree, and blooms pink flowers in March - April.
Cedar is a pyramidal shaped, evergreen tree, and blooms nondescript flowers in April - May.
```

=== Interface Implementation ===

* Core
** Interface: vanilla.Calulable
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
self  # receiver we want to make an instance of FluidForce with
```
** Java program
```java
package vanilla;

import java.util.List;
import org.jruby.embed.PathType;
import org.jruby.embed.ScriptingContainer;

public class InterfaceImplSample {
    private final String filename1 = "ruby/calculation.rb";
    private final String filename2 = "ruby/fluid_force.rb";

    private InterfaceImplSample() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer();

        // implemented by a Ruby class
        Object receiver = container.runScriptlet(PathType.CLASSPATH, filename1);
        Calculable c = container.getInstance(receiver, Calculable.class);
        List<Long> xyz = c.dimension(10L);
        System.out.format("Dimensions are %d x %d x %d.\n", xyz.get(0), xyz.get(1), xyz.get(2));
        double adjacent = 3.0;
        double opposite = 4.0;
        Double hypotenuse = c.hypotenuse(adjacent, opposite);
        System.out.format("Adjacent, opposite, and hypotenuse are %.2f, %.2f, %.2f.\n",
                adjacent, opposite, hypotenuse);

        // implemented by a top level method
        container.getVarMap().clear();
        container.put("@w", 62.4); // weight-densities of water in pounds per cubic foot
        receiver = container.runScriptlet(PathType.CLASSPATH, filename2);
        FluidForce f = container.getInstance(receiver, FluidForce.class);
        double a = 2.0;
        double b = 3.0;
        double depth = 6.0;
        Double force = f.getFluidForce(a, b, depth);
        System.out.format("Water force to %.2f ft x %.2f ft ellipse in depth of %.2f ft is %.5f lb.\n",
                a, b, depth, force);

    }

    public static void main(String[] args) {
        new InterfaceImplSample();
    }
}
```
Outputs:
```
[vanilla.InterfaceImplSample]
Dimensions are 11 x 12 x 9.
Adjacent, opposite, and hypotenuse are 3.00, 4.00, 5.00.
Water force to 2.00 ft x 3.00 ft ellipse in depth of 6.00 ft is 7057.27374 lb.
```

=== Interface Implementation with Exception and Sharing Variables ===

* Core
** Interface: vanilla.QuadraticFormula
```java
package vanilla;

import java.util.List;

public interface QuadraticFormula {
    List<Double> solve() throws RuntimeException;
}
```
** Implementation: quadratic_formula.rb
```rb
# if ax^2+bx+c=0 and b^2-4ac >=0 then
# x = (-b +/- sqrt(b^2-4ac))/2a

class QuadraticFormula
  include Java::vanilla.QuadraticFormula
  def solve()
    if $a == 0: raise RangeError end
    v = $b ** 2 - 4 * $a * $c
    if v < 0: raise RangeError end
    s0 = ((-1)*$b - Math.sqrt(v))/(2*$a)
    s1 = ((-1)*$b + Math.sqrt(v))/(2*$a)
    return s0, s1
  end
end
QuadraticFormula.new
```
** Java program
```java
package vanilla;

import java.util.List;
import org.jruby.embed.EmbedEvalUnit;
import org.jruby.embed.PathType;
import org.jruby.embed.ScriptingContainer;
import org.jruby.exceptions.RaiseException;

public class InterfaceImplSample2 {

    private final String filename = "ruby/quadratic_formula.rb";

    private InterfaceImplSample2() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer();
        EmbedEvalUnit unit = container.parse(PathType.CLASSPATH, filename);
        evaluate(container, unit, "x^2 + x - 6 = 0", 1.0, 1.0, -6.0);
        evaluate(container, unit, "3x^2 + 4x - 5 = 0", 3.0, 4.0, -5.0);
        evaluate(container, unit, "3x^2 + 4x + 5 = 0", 3.0, 4.0, 5.0);
        evaluate(container, unit, "2x^2 - 3x + 1 = 0", 2.0, -3.0, 1.0);
    }

    private void evaluate(ScriptingContainer container, EmbedEvalUnit unit,
            String expression, double a, double b, double c) {
        try {
            System.out.print("Solutions of " + expression + " are ");
            container.put("$a", a);
            container.put("$b", b);
            container.put("$c", c);
            Object receiver = unit.run();
            QuadraticFormula q = container.getInstance(receiver, QuadraticFormula.class);
            List<Double> solutions = q.solve();
            System.out.format("%.4f, and %.4f.\n", solutions.get(0), solutions.get(1));
        } catch (RuntimeException e) {
            if (e instanceof RaiseException) {
                if ("RangeError".equals(((RaiseException) e).getMessage())) {
                    System.out.println("complex.");
                }
            }
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new InterfaceImplSample2();
    }
}
```
Outputs:
```
[vanilla.InterfaceImplSample2]
Solutions of x^2 + x - 6 = 0 are -3.0000, and 2.0000.
Solutions of 3x^2 + 4x - 5 = 0 are -2.1196, and 0.7863.
Solutions of 3x^2 + 4x + 5 = 0 are complex.
ruby/quadratic_formula.rb:11:in `solve':  (RangeError)
        from :1
        ...internal jruby stack elided...
        from QuadraticFormula.solve(:1)
        from (unknown).(unknown)(:1)
Solutions of 2x^2 - 3x + 1 = 0 are 0.5000, and 1.0000.
```

=== Parse Once, Eval Many Times ===

* Core
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

** Java program
```java
package vanilla;

import org.jruby.embed.EmbedEvalUnit;
import org.jruby.embed.PathType;
import org.jruby.embed.ScriptingContainer;

public class ParseOnceEvalManyTimesSample {
    private static final String filename = "ruby/count_down.rb";

    private ParseOnceEvalManyTimesSample() {
        System.out.println("[" + getClass().getName() + "]");
        ScriptingContainer container = new ScriptingContainer();
        EmbedEvalUnit unit = container.parse(PathType.CLASSPATH, filename);
        evaluate(container, unit, 10, 11);
        evaluate(container, unit, 1, 1);
        evaluate(container, unit, 12, 31);
        evaluate(container, unit, 7, 4);
    }

    private void evaluate(ScriptingContainer container, EmbedEvalUnit unit, int month, int day) {
        container.put("@month", month);
        container.put("@day", day);
        Object ret = unit.run();
        System.out.println(ret);
        container.getVarMap().clear();
    }

    public static void main(String[] args) {
        new ParseOnceEvalManyTimesSample();
    }
}
```

Outputs:
```
[vanilla.ParseOnceEvalManyTimesSample]
Happy Birthday!
You have 82 day(s) to your next birthday!
You have 81 day(s) to your next birthday!
You have 266 day(s) to your next birthday!
```

=== Unicode Escape Support ===

```java
ScriptingContainer container = new ScriptingContainer();
Writer writer = new StringWriter();
container.setWriter(writer);
String str = "\u3053\u3093\u306b\u3061\u306f\u4e16\u754c";
container.runScriptlet("puts \"" + str + "\"");
System.out.println(writer.toString());
```

Output
```
こんにちは世界
```

== Servlet Examples ==
See [[RedBridgeServletExamples]] page.