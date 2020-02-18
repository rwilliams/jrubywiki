== require 'path/to/file.jar' ==
:require can take a path to a jar file.  It searches the JRuby ''$LOAD_PATH'' for the file.

== require 'java' ==
: A special `require 'java'` directive in your file will give you access to any bundled Java libraries (classes within your java class path).  However, this will not give you access to any non-bundled libraries.

== #each ==
: JRuby supports calling the `#each` method on Enumerable objects

== Name translations ==
:Java: org.foo.department.Widget
Ruby: Java::OrgFooDepartment::Widget

For the top-level Java packages ''java, javax, org, and com'' you can type in a fully qualified class name:

```ruby
java_import java.lang.System
```

You can use your own top-level package names by adding a ''def'':
```ruby
def edu
  Java::Edu
end

java_import edu.example.MyClass
```

== Importing a package into a class or module ==
: 
```ruby
module M
 include_package "org.xxx.yyy"
 # now any class within "org.xxx.yyy" will be available within
 # this module, ex if "org.xxx.yyy.MyClass" exists
 # a = MyClass.new
end
```
Note that there are issues with include_package, discussed at [[http://jira.codehaus.org/browse/JRUBY-5558]].

== enums ==
: Available as constants: 
```ruby
ms_constant_value = java.util.concurrent.TimeUnit::MILLISECONDS
```

== Alternative Names and Beans Convention ==

You can use underscore-separated names in addition to camel-cased names.  These two call the same method:

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

== Constructors ==

''JavaClass.new'' or ''JavaClass.new(x,y,z)'' generally work as expected. If you wish to select a particular constructor by signature use reflection:

```ruby
 # Get the the three-integer constructor for this class
 construct = JavaClass.java_class.constructor(Java::int, Java::int, Java::int)
 # Use the constructor
 object = construct.new_instance(0xa4, 0x00, 0x00).to_java
```

== java_class ==
: returns the Java class of an object.  Use this for methods expecting a java Class object:

```ruby
  DoSomethingWithJavaClass(MyJavaClass.java_class)
```


;java_kind_of? 
:works like the `instanceof` operator.

;java_object
: returns the underlying Java object. This is useful for reflection.

;java_send
: overrides JRuby's dispatch rules and forces the execution of a named Java method on a Java object. This is useful for Java methods, such as `initialize`, with names that conflict with built-in Ruby methods.

With the `java_method` method you can get a reference to any overloaded Java method as a Ruby Method object:

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

== java_method ==
: retrieves a bound or unbound handle for a Java method to avoid the reflection inherent in `java_send`.

== declared_method ==
: 

```ruby
  @mgr = javax.media.rtp.RTPManager.newInstance
  localhost = java.net.InetAddress.getByName("127.0.0.1")
  localaddr = javax.media.rtp.SessionAddress.new(localhost, 21000, localhost, 21001)
  method = @mgr.java_class.declared_method(:initialize, javax.media.rtp.SessionAddress )
  method.invoke @mgr.java_object, localaddr.java_object
```

== Type conversion ==
: See [[CallingJavaFromJRuby]]

== Arrays ==
: constructs a Java array from a Ruby array:

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

== Constructing Empty Java Arrays ==
: Use the `[]` method of the primitive types in the Java module:

```ruby
  bytes = Java::byte[1024].new # Equivalent to Java's bytes = new byte[1024];
```

== Ruby String to Java Bytes and back again ==
:
```ruby
  bytes = 'a string'.to_java_bytes
  => #<#<Class:01x9fcffd>:0x40e825 @java_object=[B@3d476c>

  string = String.from_java_bytes bytes
  => "a string"
```

== Java InputStream to a ruby IO object ==
:
```ruby
  io = input_stream.to_io # works for InputStreams, OutputStreams, and NIO Channels
```

== Interfaces ==
: 
JRuby classes can implement more than one Java interface. Use ''include'' to implement an interface:

```ruby
  class SomeJRubyObject
    include java.lang.Runnable
    include java.lang.Comparable
  end
```

When calling a method that expects an interface, JRuby checks if a block is passed and automatically converts the block to an object implementing the interface.

== Exceptions ==
: 
```ruby
 begin
   java.lang.Integer.parse_int("asdf")
 rescue java.lang.NumberFormatException => e
   puts "Failed to parse integer: #{e.message}"
 end
```

```ruby
 begin
   raise java.lang.IllegalArgumentException.new("Bad param")
 rescue java.lang.IllegalArgumentException => e
   puts "Illegal argument: #{e}"
 end
```

Or

```ruby
 begin
   raise Java::JavaLang::IllegalArgumentException.new("Bad param")
 rescue Java::JavaLang::IllegalArgumentException => e
   puts "Illegal argument: #{e}"
 end
```

== synchronize ==
: a ''synchronize'' method is provided on every wrapped Java object. For example:

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

== java_require ==
:

Normally, `jrubyc --java(c)` will include the complete source of your script in the compiled class, but often you may want to keep the source on disk or generate multiple classes that all come from the same file. To do this, add a java_require line (plus a require 'java' line to enable Java support in the Ruby code) that specifies what filename to load. The filename specified is loaded using normal Ruby 'require' semantics.

```ruby
 require 'java'
 java_require 'my_foo'
 class Foo
   def bar(a,b)
     puts a + b
   end
 end
```

== java_signature ==
: 

```ruby
 require 'java'
 java_require 'my_foo'
 class Foo
   java_signature 'void bar(int, int)'
   def bar(a,b)
     puts a + b
   end
 end
```

== java_package ==
:

```ruby
 require 'java'
 java_require 'my_foo'
 java_package 'com.example'

 class Foo; end
```

== java_annotation ==
: adds Java annotations to a JRuby class or method.

```ruby
 require 'java'
java_import 'javax.ws.rs.Path'
java_import 'javax.ws.rs.GET'
java_import 'javax.ws.rs.Produces'

java_package 'com.headius.demo.jersey'
java_annotation 'Path("/helloworld")'
class HelloWorld
  java_annotation 'GET'
  java_annotation 'Produces("text/plain")'
  def cliched_message
    "Hello World"
  end
end
```