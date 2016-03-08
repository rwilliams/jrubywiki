Or how to create ruby methods using java see [jruby-examples][]

## Class method signatures 
```ruby
# in ruby

def active?
end
```
```java
// java, note active_p follows the naming convention for a query

@JRubyMethod(name = "active?")
public IRubyObject active_p(ThreadContext context) {
}
```
```ruby
# in ruby we might have vector normalize

def normalize
end
```
```java
// java, norm we will return a normalized copy leaving original unchanged

@JRubyMethod(name = "normalize")
public IRubyObject norm(ThreadContext context) {
}
```
```ruby
# in ruby we might have vector normalize, where we change the original

def normalize!
end
```
```java
// in java, the normalize_bang follows the naming convention for a hard change
// eg in vector normalize the original is changed

@JRubyMethod(name = "normalize!")
public IRubyObject norm_bang(ThreadContext context) {
}
```
If you are creating your own objects it not a bad idea to create a String representation that can be used by inspect for example (regular ruby will do something sensible for this) eg:-
```java
  /**
  * @param context ThreadContext
  * @return IRubyObject to_s
  */
  @JRubyMethod(name = {"to_s", "inspect"})
  
  public IRubyObject to_s(ThreadContext context) {
    return context.getRuntime().newString(String.format("Vec3D(x = %4.4f, y = %4.4f, z = %4.4f)", jx, jy, jz));
  }
```
In the above case had we not been aliasing the `to_s` to `inspect`, we could have omitted name from the @JRubyMethod annotation and the ruby method name would be taken from the name of the java method.

### rest arguments
```ruby
# ruby

def initialize(*args)
end
```
```java
// java

@JRubyMethod(name = "initialize", rest = true)
public IRubyObject initialize(ThreadContext context, IRubyObject[] args) {
}
```
### arguments with default value
```ruby
# ruby

def get(key, marshal=true)
end
```
```java
// java (the default value for required and optional is 0)

@JRubyMethod(name = "get", required = 1, optional = 1)
public IRubyObject get(ThreadContext context, IRubyObject[] args) {
    Ruby ruby = context.getRuntime();
    RubyString key = (RubyString) args[0];
    RubyBoolean marshal = ruby.getTrue();
    if (args.length > 1) {
        marshal = args[1];
    }
}
```

### initialize or new
```ruby
# ruby

attr_reader :x, :y

def initialize(*args)
  if args.length == 2
    @x = args[0]
    @y = args[1]
  else
    @x = 0
    @y = 0
  end
end
```
Note the `new` method should be a class method, so it is a static method in Java. Also it is `meta` method that should be defined on the metaclass (hence `meta = true` in the @JRubyMethod annotation). The convention is to name the java method `rbNew`, this is perhaps one the trickiest methods to implement correctly, where you will most likely create some helper methods as we have below. Note we can possibly should do an arity check on args, klazz is the class we are creating (as with module methods below static methods expect a receiver).
```java
// java where meta = true mean this method should be defined on the metaclass

/**
 *
 * @param context ThreadContext
 * @param klazz IRubyObject
 * @param args optional (no args jx = 0, jy = 0)
 * @return new Vec2 object (ruby)
 */
@JRubyMethod(name = "new", meta = true, rest = true)
public static final IRubyObject rbNew(ThreadContext context, IRubyObject klazz, IRubyObject[] args) {
    Vec2 vec2 = (Vec2) ((RubyClass) klazz).allocate();
    vec2.init(context, args);
    return vec2;
}

/**
 *
 * @param runtime Ruby
 * @param klass RubyClass
 */
public Vec2(Ruby runtime, RubyClass klass) {
    super(runtime, klass);
}

void init(ThreadContext context, IRubyObject[] args) {
    if (Arity.checkArgumentCount(context.getRuntime(), args, Arity.OPTIONAL.getValue(), 2) == 2) {
        jx = (Double) args[0].toJava(Double.class);
        jy = (Double) args[1].toJava(Double.class);
    }
}
```

## Module method signatures
```ruby
module Foo 
  def build_string
     return 'This is a new String' 
  end 
  alias_method :build_string :new_string
end
```
Note in java the module method is a static method, and requires a receiver, also illustrated below is how to create an alias (there is an alias option in jruby annotations but it is not required just `name` the method as below
```java
/**
*
* @param context ThreadContext
* @param recv the receiver
* @return A RubyString.
*/
@JRubyMethod(name = {"build_string", "new_string"}, module = true)
public static IRubyObject buildString(ThreadContext context, IRubyObject recv) {
    Ruby runtime = context.getRuntime();
    return runtime.newString("This is a new String");
}
```
Should you wish to create a module class method (as below) 
```ruby
module Foo 
  def self.build_string
     return 'This is a new String' 
  end
end
```
We create the class method using the @JRubyMethod annotation `meta = true` which directs that this method should be defined on the metaclass.
```java
/**
*
* @param context ThreadContext
* @param recv the receiver
* @return A RubyString.
*/
@JRubyMethod(name = {"build_string", "new_string"}, module = true, meta = true)
public static IRubyObject buildString(ThreadContext context, IRubyObject recv) {
    Ruby runtime = context.getRuntime();
    return runtime.newString("This is a new String");
}
```

[jruby-examples]:https://github.com/jruby/jruby-examples