<h1>JRuby and Java Code Examples</h1>
Below are some code examples showing how to call JRuby from Java and how to call Java from JRuby.<br/><br/>

__TOC__
==JRuby calling Java==

See also: [[CallingJavaFromJruby|Calling Java from JRuby]]

===JRuby: <tt>call_java.rb</tt>===
```ruby
require "java"

java_import "java.util.TreeSet"
java_import "com.example.CallMe"
java_import "com.example.ISpeaker"

puts "Hello from ruby"
set = TreeSet.new
set.add "foo"
set.add "Bar"
set.add "baz"
set.each { |v| puts "value: #{v}" }

cm = CallMe.new
cm.hello
$globalCM.hello

class CallJava
  include ISpeaker
  def initialize
    super
    @count = 0
  end

  def say(msg)
    puts "Ruby saying #{msg}"
  end
  
  def addOne(from)
#    m.synchronize {
      @count += 1
      puts "Now got #@count from #{from}"
#    }
  end
end
```
===Java: <tt>ISpeaker.java</tt>===
```java
package com.example;

public interface ISpeaker {
    public void say(String msg);
    
    public void addOne(String from);
}
```
===Java: <tt>CallMe.java</tt>===
```java
package com.example;

public class CallMe {

    String mName;

    public CallMe() {
        this("Default");
    }
    
    public CallMe(String name) {
        mName = name;
    }
    
    public void hello() {
        System.out.println("Hello from "+mName);
    }
    
    public static void main(String []args) {
        System.out.println("Called main");
    }
}
```

==Java calling JRuby==

See the following pages for updated Java embedding documentation:

[[RedBridge]] - The RedBridge API is the recommended Java API to use when embedding JRuby into a Java application or writing Java code to drive JRuby.

[[Embedding-with-JSR-223|Embedding with JSR-223|]] - JSR-223, also known as the the `javax.script` API, is a standard Java API for driving embedded JVM languages. It is not as direct as RedBridge but it is standard across all JVM languages that support it.