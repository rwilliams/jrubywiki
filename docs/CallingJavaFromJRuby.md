Calling Java from JRuby
=======================

All the following examples can be "run" either from the command line, or by using **jirb_swing**, the Swing-based IRB console that comes with JRuby.  You can start jirb_swing like `$ jruby -S jirb_swing`.

A special `require 'java'` directive in your file will give you access to any bundled Java libraries (classes within your java class path).  If you need to access Java libraries not contained with the Java class path we will show you how to do that in a later section.

The following code shows typical use. It should pop up a small window showing "Hello" on your screen.

```ruby
# This is the 'magical Java require line'.
require 'java'

# With the 'require' above, we can now refer to things that are part of the
# standard Java platform via their full paths.
frame = javax.swing.JFrame.new("Window") # Creating a Java JFrame
label = javax.swing.JLabel.new("Hello")

# We can transparently call Java methods on Java objects, just as if they were defined in Ruby.
frame.add(label)  # Invoking the Java method 'add'.
frame.setDefaultCloseOperation(javax.swing.JFrame::EXIT_ON_CLOSE)
frame.pack
frame.setVisible(true)
```

**Note:** If you are testing the example above in the Swing IRB console `jirb_swing`, change the default close operation to `DISPOSE_ON_CLOSE`, or `HIDE_ON_CLOSE` unless you want `jirb_swing` to also close when you close the second window.

Here's another example (showing results from testing these statements in the `jirb` console). 

Let's say you wanted to get a list of network interfaces. You can get Java API docs at [java.net.NetworkInterface](http://java.sun.com/j2se/1.5.0/docs/api/index.html).

Here's how to access the methods from this Java Class from from JRuby:

```ruby
  irb(main):013:0> ni = java.net.NetworkInterface.networkInterfaces
  => #<#<Class:01x7e666f>:0x855a27 @java_object=java.net.NetworkInterface$1@821453>
```

**`ni`** is a Ruby variable holding a Java Enumeration of NetworkInterfaces. You can see the Class ancestry for `ni` like this:

```ruby
  irb(main):029:0> ni.class.ancestors
  => [#<Class:01x7e666f>, Java::JavaUtil::Enumeration, Enumerable, Java::JavaLang::Object, ConcreteJavaProxy, JavaProxy, JavaProxyMethods, Object, Java, Kernel]
```

Enumeration elements can't be accessed using `Array#[]` syntax but they do appear as Arrays for many other purposes. You can find out both the Java and Ruby methods for an Enumeration of NetworkInterfaces like this:

```ruby
  irb(main):032:0> java.net.NetworkInterface.networkInterfaces.methods
  => ["__jsend!", "has_more_elements", "hasMoreElements", "next_element", "nextElement",
  "each", "reject", "member?", "grep", "include?", "min", "sort", "any?", "partition", 
  "each_with_index", "collect", "find_all", "to_a",  "inject", "detect", "map", "zip", 
  "sort_by", "max", "entries", "all?", "find", "select", ...]
```

You could also call #to_a then use Array#[] on them, etc.

Because JRuby supports the `#each` method on Java Enumerations you could also do this:

```ruby
  irb(main):011:0> java.net.NetworkInterface.networkInterfaces.each {|i| puts i; puts }
  name:en1 (en1) index: 5 addresses:
  /63.138.152.170;
  /fe80:0:0:0:21b:63ff:febf:4a9d%5;
  
  name:en0 (en0) index: 4 addresses:
  /63.138.152.125;
  /fe80:0:0:0:21b:63ff:fe1e:b2da%4;
  
  name:lo0 (lo0) index: 1 addresses:
  /127.0.0.1;
  /fe80:0:0:0:0:0:0:1%1;
  /0:0:0:0:0:0:0:1%0;
```

Accessing and Importing Java Classes
------------------------------------

### From jar files

To use resources within a jar file from JRuby, the jar file must either be on the classpath *or* be made available with the `require` method:

```ruby
require 'path/to/mycode.jar'
```
This `require` makes the resources in `mycode.jar` discoverable by later commands like `import` and `include_package`.

Note that loading jar-files via `require` searches along the `$LOAD_PATH` for them, like it would for normal ruby files.

### From .class files

If you need to load from an existing .class file (or one that's not camel-case), the following has examples: [[http://www.ruby-forum.com/topic/216572#939791]]

Basically it's `$CLASSPATH << "target/classes"; java_import org.asdf.ClassName` where "target/classes/org/asdf/ClassName.class" exists.

Note also that this will need to be a full path name or relative to the directory the script starts in, as the JVM doesn't seem to respond to Dir.chdir very well.

Referencing Java Classes (using full-qualified class name)
------------------------

You can reference a Java class in JRuby in at least two different ways. 

* The first is to map Java names like `org.foo.department.Widget` to Ruby nested modules format. This works as follows:

```
Java: org.foo.department.Widget
Ruby: Java::OrgFooDepartment::Widget
```

That is:

* Java packages all reside within the `Java` module.
* The package path is then transformed by removing the dots and converting to _CamelCase_

This also means that, just as in Java, packages are not nested, but are each associated with their own unique module name.

* Second way: for the top-level Java packages `java`, `javax`, `org`, and `com` you can type in a fully qualified class name basically as in Java, for example, `java.lang.System` or `org.abc.def.ClassName`
You can get the same effect for your own (custom) top-level packages, as follows. Let's assume that your packages are called `edu.school.department.Class`. Then, you define

```ruby
def edu
  Java::Edu
end
```

And then you can use usual Java package names like `edu.school.department.Class`
Note also that you must use the right capitalization, though see below for how you can change and assign it to your own liking.

Using a Java Class Without The Full Path Name
---------------------------------------------

You can always access any Java class that has been loaded or is in the classpath by specifying its full name (e.g. java.lang.System). With the `java_import` statement, you can access the Java class via a constant that java_import creates for you in the current namespace.  Let's consider how to make using `java.lang.System` easier to use:

**Note:** For older scripts where `import` was used we have deprecated that for `java_import` to not conflict with rake.

```ruby
  require 'java'
  java_import java.lang.System
  version = System.getProperties["java.runtime.version"]
```

In this script when java_import executes it will make a new constant for you called `System`.  As this is happening in a top-level script it will be put into ::Object.  Now in the next line you can just use `System`.

As this is just a constant you need to be wary of redefining or papering over other constants.  Consider an example like this:

```ruby
require 'set'
java_import java.util.Set
```

In this case you defined the constant `Set` on `::Object` from Ruby's set library and then you overwrote the constant `Set` on `::Object` by calling `java_import`.  Luckily, in this case you will see a warning telling you that you changed a constant definition.  This example also gives a cautionary tale.

Java and Ruby classes have an overlap on naming.  So if you include a Java Class at a top-level scope it can mess up the world of Ruby since what you thought was a Ruby set suddenly becomes a Java one.  Ruby's constant resolution is much more tricky than it seems so reviewing an article on [constant resolution](http://valve.github.io/blog/2013/10/26/constant-resolution-in-ruby/) may help you to better understand how constants work.  Our recommendation is to namespace your java_imports into the non-top-level namespace in a Ruby program of any size.

A more confusing example is when you unintentionally wallpaper over a constant:

```ruby
require 'set'

module MyClass
  java_import java.util.Set

  Set # 1. ???
end

Set # 2. ???
```

The `java_import` will define a `Set` on `MyClass` (e.g. `MyClass::Set`).  If you access `Set` at 1 you will get this new constant.  If you access `Set` at 2 you will get `Object::Set`.  Both of these examples are just cases how constants in Ruby work but the implicit constant creation of java_import can trip programmers up.

It is also possible to call java_import from within methods and the rules for java_import is intended to add the constant to the class it is being called on (class method is self and instance method is self.class).  We however recommend against doing this as it is not as intuitive as we had hoped.  Try and import in module or class bodies like in the examples above.

Some people do not like the implicit side-effect nature of `java_import`, so you can also get the same effect by reassigning a Java class to a new constant, like

```ruby
  require 'java'
  Sys = java.lang.System
  version = Sys.getProperties["java.runtime.version"]
```

Use `include_package` within a Ruby Module to import a Java Package's classes on const_missing
-----------------------------------------

Use `include_package "package_name"` in a Ruby Module to support namespaced access to the Java classes in the package. This is similar to Java's `package xxx.yyy.zzz;` format.  It is also legal to use `import "package_name"`, that is similar to `import package_name.*`.

**Example**: using `include_package` in a module

```ruby
module MyApp
 include_package 'org.apache.logging.log4j'
 # now any class from "org.apache.logging.log4j" will be available within
 # this module, ex if "org.apache.logging.log4j.LogManager" exists ...
 Logger = LogManager.getLogger('MyApp')
end
```

**Example**: create a Ruby Module called `JavaLang` that includes the classes in the Java package `java.lang`.
```ruby
  module JavaLangDemo
    include_package 'java.lang'
    # alternately, use the #import method
    #import 'java.lang'
  end
```

Now you can prefix the desired Java Class name with `JavaLangDemo::` to access the included Classes:

```ruby
  version = JavaLangDemo::System.out # java.lang.System.out
  => #<Java::JavaIo::PrintStream:0x30a7202>

  processors = JavaLangDemo::Runtime.getRuntime.availableProcessors
  => 2
```

All Java classes in the package will become be available in this class/module, unless a constant with the same name as a Java class is already defined.

The use of the Module name to scope access to the imported Java class is also helpful in cases where the Java class has the same name as an existing Ruby class. 

For example if you need to create an instance of a `java.io.File` object, this code will work:

```ruby
  java_import java.io.File
  newfile = File.new("file.txt")
  => #<Java::JavaIo::File:0xdc6f00 @java_object=file.txt>
```

However you've now redefined the Ruby constant File and can no longer access the Ruby File class. Executing this:

```ruby
  File.open('README', 'r') {|f| puts f.readline }
```

Will produce this error:

```ruby
  NoMethodError: private method `open' called for Java::JavaIo::File:Class
```

If instead you create a module called `JavaIO` and include the package in the module definition:

```ruby
  module JavaIO
    include_package "java.io"
  end
```

You can now create a new instance of the Java class File without shadowing the Ruby version of the File class:

```ruby
  newfile = JavaIO::File.new("file.txt")
  => #<Java::JavaIo::File:0x15619c @java_object=file.txt>
```

And the Ruby File class is still accessible:

```ruby
  File.open('README.md', 'r') {|f| puts f.readline }
  JRuby -  A Java implementation of the Ruby language
  => nil
```

Making them accessible in the main name space
---------------------------------------------

You can set `const_missing` to make them accessible in more than just a module. See [[http://www.ruby-forum.com/topic/216629]] though there might be a better way.

Using within classes within module
----------------------------------

Currently, `include_package` lookup doesn't work within any sub-modules or sub-classes e.g. running : 

```ruby
  module GUI
    include_package 'javax.swing'
    include_package 'java.awt.image' # BufferedImage
    # bellow won't work without accessing BufferedImage before :
    class ShowImage < JFrame
      BufferedImage.new(10, 20, BufferedImage::TYPE_BYTE_BINARY)
    end
  end
```
```
  NameError: uninitialized constant GUI::ShowImage::BufferedImage
	from org/jruby/RubyModule.java:2743:in `const_missing'
	from (irb):6:in `ShowImage'
	from (irb):5:in `GUI'
	from (irb):1:in `evaluate'
```

You could override `const_missing`, though (see above), if you wanted that behavior.

Using Static Java Enumerations
------------------------------

Java `enum`s are accessible from Ruby code as constants:

```ruby
  lock.try_lock(5000, java.util.concurrent.TimeUnit::MILLISECONDS)
```

or

```ruby
  java_import java.util.concurrent.TimeUnit
  lock.try_lock(5000, TimeUnit::MILLISECONDS)
```

Gotchas
-------

JRuby automatically binds the following names in the context of a class to the top-level Java packages: `com`, `org`, `java`, `javax` and `javafx`. This means that you can reference these packages without having to explicitly require or import them. This takes effect for all Ruby classes in an application where a `require 'java'` appears. This binding takes place in precedence to the classes *method_missing* handling.

If you do not want this behaviour for a specific class, you can undefine it for that class. Here's an example that will execute identically under Ruby and JRuby:

```ruby
# Note: Comment out for Ruby test. Uncomment for JRuby test.
require 'java'

class MethodMissing
  # JRuby: Undefine the standard "automatic" bindings to Java, to avoid any automatic binding.
  undef org, com, java, javax, javafx
  
  def method_missing(m, *args)
    puts "method_missing: #{m}."
    "This is what I return from method missing."
  end
end

mm = MethodMissing.new
result = mm.org
puts "Result: #{result} Type: #{result.class.name}"
```

Calling a Java Method
---------------------

Alternative Names and Beans Convention
--------------------------------------

In Ruby, one usually prefers `method_names_like_this`, while Java traditionally uses `methodNamesLikeThis`. If you want, you can use Ruby-style method names instead of the Java ones.

For example, these two calls are equivalent

```ruby
  java.lang.System.currentTimeMillis
  java.lang.System.current_time_millis
```

JRuby also translates methods following the 'beans-convention':

```ruby
  x.getSomething            becomes   x.something
  x.setSomething(newValue)  becomes   x.something = new_value
  x.isSomething             becomes   x.something?
```

You don't have to use these alternatives, but they can make the interaction with Java code feel more Ruby-like.

Constructors
------------

`JavaClass.new` or `JavaClass.new(x,y,z)` generally works as expected. If you wish to select a particular constructor by signature use reflection:

```ruby
 # Get the the three-integer constructor for this class
 construct = JavaClass.java_class.constructor(Java::int, Java::int, Java::int)
 # Use the constructor
 object = construct.new_instance(0xa4, 0x00, 0x00)
```

Beware of Java generics
-----------------------

If a Java class is defined with Java generics, the types are erased during compilation for backwards compatibility. As a result, JRuby will have problems with automatic type conversion. For example, if you have a `Map<String,String>`, it will be seen as a simple `Map`, and JRuby will not be able to determine the correct types using reflection.

Additional Java Method Access
-----------------------------

JRuby defines a number of additional methods for Java objects.

* `java_class` returns the Java class of an object.
* `java_kind_of?` works like the `instanceof` operator.
* `java_object` returns the underlying Java object. This is useful for reflection.
* `java_send` overrides JRuby's dispatch rules and forces the execution of a named Java method on a Java object. This is useful for Java methods, such as `initialize`, with names that conflict with built-in Ruby methods (more on `java_send` below).
* `java_method` retrieves a bound or unbound handle for a Java method to avoid the reflection inherent in `java_send`  (more on `java_method` below).
* `java_alias` aliases a specific method name and signature to a new Ruby method name

Calling masked or unreachable Java methods with `java_send`
-----------------------------------------------------------

Sometimes you need to call `initialize` AFTER the `.new()` call, for example the `RTPManager` class in JMF. Unfortunately, this method is masked by Ruby's `initialize` constructor method.  As of JRuby 1.4, the `java_send` method can be used to call this, and any other, masked method:

```ruby
  @mgr = javax.media.rtp.RTPManager.newInstance
  localhost = java.net.InetAddress.getByName("127.0.0.1")
  localaddr = javax.media.rtp.SessionAddress.new(localhost, 21000, localhost, 21001)
  @mgr.java_send :initialize, [javax.media.rtp.SessionAddress], localaddr
```

Here is another example of calling the `ArrayList.add` method with `java_send`:

```ruby
 java_import java.util.ArrayList
 list = ArrayList.new
 list.java_send :add, [Java::int, java.lang.Object], 0, 'foo'
 puts list.java_send :toString # => "[foo]"
```

Note the second argument, which is an array of types indicating the exact method signature desired. This is useful for disambiguating methods that are overloaded on similar types such as `int` and `long`.

Bound and Unbound Java methods with `java_method`
-------------------------------------------------

`java_send` relies on reflection and may lead to poor performance in some cases. Each time it is called, the desired method must be relocated. With the `java_method` method you can get a reference to any overloaded Java method as a Ruby Method object:

```ruby
 # get a bound Method based on the add(int, Object) method from ArrayList
 add = list.java_method :add, [Java::int, java.lang.Object]
 add.call(0, 'foo')
```

Similarly, an Unbound method object can be retrieved:

```ruby
 # get an UnboundMethod from the ArrayList class:
 toString = ArrayList.java_method :toString
 toString.bind(list).call # => [foo, foo]
```

_Note:_ When specifying parameters to java_method, you access primitive data types via `Java::`, i.e. `Java::char`, `Java::int`.   On the other hand, Classes must be specified directly. `java.lang.String`, `java.io.InputStream`, etc.

Aliasing a specific method with `java_alias`
--------------------------------------------

If you'll be calling a specific unreachable Java method a lot, or if a Java method has many overloads and you want to avoid the overhead of selecting the right one every time, you can use `java_alias` to alias a specific Java method name + signature to a Ruby name.

```ruby
 java_import java.util.ArrayList
 class ArrayList
   java_alias :simple_add, :add, [Java::int, java.lang.Object]
 end
 list = ArrayList.new
 list.simple_add 0, 'foo'
```

Reflection
----------

Here is an example that shows using java's reflection within jruby

```ruby
  @mgr = javax.media.rtp.RTPManager.newInstance
  localhost = java.net.InetAddress.getByName("127.0.0.1")
  localaddr = javax.media.rtp.SessionAddress.new(localhost, 21000, localhost, 21001)
  method = @mgr.java_class.declared_method(:initialize, javax.media.rtp.SessionAddress )
  method.invoke @mgr.java_object, localaddr.java_object
```

Conversion of Types
-------------------

Ruby to Java
------------

_See the JRuby rspec source code dir_ `spec/java_integration` _for many more examples._

When calling Java from JRuby, primitive Ruby types are converted to default boxed Java types:

<table>
	<tr>
		<th>Ruby Type</th>
		<th>Java Type</th>
	</tr>
	<tr>
		<td>"foo"</td>
		<td>java.lang.String</td>
	</tr>
	<tr>
		<td>1</td>
		<td>java.lang.Long</td>
	</tr>
	<tr>
		<td>1.0</td>
		<td>java.lang.Double</td>
	</tr>
	<tr>
		<td>true, false</td>
		<td>java.lang.Boolean</td>
	</tr>
	<tr>
		<td>1 << 128</td>
		<td>java.math.BigInteger</td>
	</tr>
</table>

However, this does not mean that you cannot call methods expecting a primitive type. You can also pass an integer to a method expecting a double value. JRuby usually tries quite hard to find a method that can understand your parameters.

If JRuby cannot find a matching method, it tries to pass the actual JRuby objects instead (that is, the Java objects from the JRuby implementation). A consequence of this is that if this fails you will see an error message stating that JRuby hasn't found a method taking an object of class `org.jruby.RubyObject` instead of the actual type.

If JRuby is not finding the exact method you want to call, perhaps because of extreme ambiguity like `foo(int)` vs. `foo(long)`, the `java_send` method can be used to disambiguate. See below.

Java to Ruby
------------

When primitive Java types are passed to JRuby they are converted to the following Ruby types:

<table>
	<tr>
		<th>Java Type</th>
		<th>Ruby Type</th>
	</tr>
	<tr>
		<td>public String</td>
		<td>String</td>
	</tr>
	<tr>
		<td>public byte</td>
		<td>Fixnum</td>
	</tr>
	<tr>
		<td>public short</td>
		<td>Fixnum</td>
	</tr>
	<tr>
		<td>public char</td>
		<td>Fixnum</td>
	</tr>
	<tr>
		<td>public int</td>
		<td>Fixnum</td>
	</tr>
	<tr>
		<td>public long</td>
		<td>Fixnum</td>
	</tr>
	<tr>
		<td>public float</td>
		<td>Float</td>
	</tr>
	<tr>
		<td>public double</td>
		<td>Float</td>
	</tr>
</table>

The Java Booleans true and false are coerced to the Ruby singleton classes TrueClass and FalseClass which are represented in Ruby with the instances true and false.

The `null` Java object is coerced to the Ruby class `NilClass` which is represented in Ruby as the instance `nil`.

Java Primitive Classes
----------------------

Java primitive classes can be found in the Java module. For example, `Java::byte` represents the primitive type byte in java. You can get its class as follows:

<table>
	<tr>
		<th>Ruby Code</th>
		<th>Java Class</th>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("byte")</td>
		<td>Java::byte.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("boolean")</td>
		<td>Java::boolean.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("byte")</td>
		<td>Java::byte.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("short")</td>
		<td>Java::short.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("char")</td>
		<td>Java::char.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("int")</td>
		<td>Java::int.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("long")</td>
		<td>Java::long.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("float")</td>
		<td>Java::float.java_class</td>
	</tr>
	<tr>
		<td>Java::JavaClass.for_name("double")</td>
		<td>Java::double.java_class</td>
	</tr>
</table>

Arrays
------

There are two ways of constructing Java arrays. One is to use the `to_java` method of the class Array. The other is to use the `[]` method for the primitive Java types.

Converting a Ruby Array to a Java Array

The `to_java` method constructs a Java array from a Ruby array:

```ruby
  [1,2,3].to_java
  => [Ljava.lang.Object;@1a32ea4
```

By default, `to_java` constructs `Object` arrays. You can specify the parameter with an additional argument which can either be a symbol or a primitive class like `Java::double`

```ruby
  ["a","b","c"].to_java(:string)
  => [Ljava.lang.String;@170984c

  [1, 2, 3.5].to_java Java::double
  => [D@9bc984
```

Constructing Empty Java Arrays
------------------------------

Sometimes a Java library will need a fixed-length array, say for example a byte buffer for a stream to read into. For this, you can use the `[]` method of the primitive types in the Java module:

```ruby
  bytes = Java::byte[1024].new # Equivalent to Java's bytes = new byte[1024];
  strings = java.lang.String[1024].new
```

Ruby String to Java Bytes and back again
----------------------------------------

```ruby
  bytes = 'a string'.to_java_bytes
  => #<#<Class:01x9fcffd>:0x40e825 @java_object=[B@3d476c>

  string = String.from_java_bytes bytes
  => "a string"
```

More Conversions
-------------------------------------------------------------

### `java.io.InputStream` to/from `IO`

```ruby
  io = java_input_stream.to_io # works for InputStreams, OutputStreams, and NIO Channels
  stream = io.to_inputstream # also to_outputstream and to_channel
```

Note that closing a stream will close its conversions.

### `java.util.Set` to `Set`
```ruby
set = java.util.HashSet.new([1,2,3])
set.to_set
=> #<Set: {1, 2, 3}>
```

### `java.util.Map` to `Hash`
```ruby
hash = java.util.HashMap.new(:a => 123) # or Hashtable
hash.to_hash.class
=> Hash
```

### `java.util.List` to `Array`
```ruby
list = java.util.ArrayList.new([1,2,3])  # or Vector, LinkedList, ...
list.to_a
=> [1, 2, 3]
```

Note that `list.to_array` will return a *primitive java array*.

### `Time` to `java.util.Date`
```ruby
t = Time.now
t.to_java
=> #<Java::JavaUtil::Date:0x747541f8>
```

Gotchas
-------

Note that currently you cannot call java methods from within a ruby class' constructor that inherits from a java class, without first calling
  super
or it results in a stack overflow http://www.ruby-forum.com/topic/217475


Referencing a `java.lang.Class` object
--------------------------------------

If you call a Java class from JRuby and need to pass a Java class as an argument, if you use this form:

```ruby
  DoSomethingWithJavaClass(MyJavaClass.class)
```

you'll get this error:

```ruby
  TypeError: expected [java.lang.Class]; 
  got: [org.jruby.RubyClass]; error: argument type mismatch
```

Instead use the method `java_class`.

```ruby
  DoSomethingWithJavaClass(MyJavaClass.java_class)
```

Integrating JRuby and Java Classes and Interfaces
-------------------------------------------------

Classes
-------

Reopening Java Classes
----------------------

In Ruby, classes are always open, which means that you can later add methods to existing classes. This also works with Java classes.

This comes in handy when adding syntactic sugar like overloaded operators to Java classes, or other methods to make them behave more Ruby-like.

Note that these additions will only be visible on the JRuby side. 

Subclassing a Java class
------------------------

You can subclass (i.e. extend) a Java class and then use the JRuby class whenever Java expects the superclass.

Creating anonymous classes
--------------------------
```ruby
foo = Class.new(Java::my.package.MyClass) {
  def method1(x)
    # override method1
  end
  def method2(y, z)
    # override method2
  end
  # etc...
}.new(my_args)
```

Accessing package, protected and private fields
----------------------------------------------------------------------

If you need to access package, protected and private fields on a Java class, you are able to do so with field_reader, field_writer and field_accessor attributes.

The methods field_reader, field_writer, field_accessor are analogues to attr_reader, attr_writer, and attr_accessor and take a symbol name of the field you want to access as a an argument. 

For example, if you have a java class of:

```java
public class Something {

 protected float somevalue = 1.0f;

}
```

And you want to access the protected field `somevalue` from a subclass that is defined in JRuby, you can do the following:

```ruby
class SomethingElse < Something
  field_accessor :somevalue

  def initialise
    self.somevalue(2.0)
  end

end
```

Gotchas
-------

If you have a class name ambiguity between Java and Ruby, the class name will reference the Ruby construct within the Ruby code. For instance, if you import `java.lang.Thread` and then write `JThread < Thread`, `JThread` will in fact inherit the Ruby `Thread` object, not the Java `Thread`. The solution is to use the full Java Class name, such as:
 `JThread < java.lang.Thread`

Interfaces
----------

Java interfaces are mapped to modules in JRuby. This means that you can also reopen the corresponding module and add further methods on the JRuby side.

Implementing Java Interfaces in JRuby
-------------------------------------
JRuby classes can now implement more than one Java interface. Since Java interfaces are mapped to modules in JRuby, you implement them not by subclassing, but by mixing them in.

```ruby
  class SomeJRubyObject
    include java.lang.Runnable
    include java.lang.Comparable
  end
```

Closure Conversion
------------------

JRuby sports a _closure conversion_ feature (a.k.a. proc-to-interface conversion), where a Ruby block (closure) is converted to an appropriate Java interface. For example:

```ruby
  button = javax.swing.JButton.new "Press me!"
  button.add_action_listener {|event| event.source.text = "You did it!"}
```

In this example, `JButton`'s `addActionListener` method takes one parameter, a `java.awt.event.ActionListener`. The block is converted to a `Proc` object, which is then decorated with a java interface proxy that invokes the block for any method called on the interface.

This works for any interface, but fits best when implementing functional-style interfaces (those that contain one method to be implemented).

In (rare) cases where there are multiple matching Java methods with different interface last parameters, the block arity is taken into consideration. Here's an example of the [overloaded](http://docs.oracle.com/javase/7/docs/api/java/io/File.html#listFiles%28java.io.FilenameFilter%29) `java.io.File#listFiles` method:
```ruby
java.io.File.new('.').listFiles do |pathname|
  # matches listFiles( FileFilter#accept(File) )
end
```
```ruby
# listFiles( FilenameFilter#accept(File, String) )
java.io.File.new('.').listFiles do |dir, name|
  # calls listFiles( FilenameFilter#accept(File, String) )
end
```
**Note**: This [feature](http://jewelruby.kares.org/an-obvious-java-dispatch) is available since JRuby **1.7.22** and **9.0.1.0**.



Java classes can't inherit from a JRuby class
---------------------------------------------

Hopefully this feature will be added in the planned re-write of the Java integration layer in a future release of JRuby.

Exception Handling
------------------

Native Java exceptions can be caught in Ruby code as expected:

```ruby
 begin
   java.lang.Integer.parse_int("asdf")
 rescue java.lang.NumberFormatException => e
   puts "Failed to parse integer: #{e.message}"
 end
```
_Note: Java exceptions do not inherit from ruby Exception, but they will be caught by `rescue`_

Furthermore, Ruby code can throw Java exceptions:

```ruby
 begin
   raise java.lang.IllegalArgumentException.new("Bad param")
 rescue java.lang.IllegalArgumentException => e
   puts "Illegal argument: #{e}"
 end
```

This is useful if you happen to be implementing a Java interface in Ruby that requires a particular exception to be thrown on error.

Note that this can also be written:

```ruby
 begin
   raise Java::JavaLang::IllegalArgumentException.new("Bad param")
 rescue Java::JavaLang::IllegalArgumentException => e
   puts "Illegal argument: #{e}"
 end
```

Synchronization in JRuby
------------------------

When interacting with Java APIs from JRuby, it is occasionally necessary to synchronize on an object for thread safety. In JRuby, a `synchronized` method is provided on every wrapped Java object to support this functionality. For example, the following Java code:

```java
 synchronized(obj) {
     obj.wait(1000); 
 }
```

is implement like this in Ruby:

```ruby
 obj.synchronized do
   obj.wait 1000
 end
```

The expression evaluates to the result of the block, e.g.,

```ruby
 obj.synchronized { 99 }  # => 99
```

Converting a java Object to a non primitive class
-----------------------------------------------
There can be instances (in closures for example) where you need to convert a java Object to a specific class/interface you can do this using a custom `to_java` eg 
```ruby
{ |obj| d = obj.to_java(Java::Hype::HDrawable); d.stroke(100) ..etc }
```
You may find that this is preferable to creating an anonymous class when implementing a simple (one method) interface, especially since jdk8 has lambda support, and even java people have seen the light.


[Java Integration **changes in JRuby 9.1**](JRuby-9.1.0.0-Release-Notes#java-integration)

[Java Integration **changes in JRuby 9.2**](JRuby-9.2.0.0-Release-Notes#java-integration)


Related Articles
----------------

* [[FAQs#calling-into-java]] Other useful examples
* [Java Integration in JRuby](http://mikiobraun.blogspot.com/2008/11/java-integration-in-jruby.html) Short overview article on JRuby and Java integration
* [JRuby/Ruby Arrays to Hash](http://kenai.com/projects/jruby/pages/CallingJavaFromJRuby) Review on Converting Arrays to Hash in JRuby/Ruby
* [Ruby Arrays and Java Lists](http://jewelruby.kares.org/java-streams-meet-enumerator)
* [Java Streams Meet Enumerator](http://jewelruby.kares.org/ruby-arrays-and-java-lists)
* [Better Closures for (Functional) Java](http://jewelruby.kares.org/better-closures-for-functional-java)
* [Java Integration: JavaScript, Groovy and JRuby](https://blogs.oracle.com/sundararajan/entry/java_integration_javascript_groovy_and) Side-by-side comparison of Java integration in JavaScript, Groovy and JRuby.
* [Spicing up Java with Ruby (a Rhino Story)](http://log.kares.org/2012/09/spicing-up-java-with-ruby-rhino-story.html) Using Java integration to bring Rhino into Ruby.
* [JtestR](http://jtestr.codehaus.org/) A testing framework for Java based on JRuby.
* [Sparrow](http://github.com/leandrosilva/sparrow) A JMS client for messaging based on JRuby.