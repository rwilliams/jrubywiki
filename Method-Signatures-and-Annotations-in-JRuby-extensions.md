## Class method signatures (see below for Module method signatures)
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