If you were directed to this page by a JRuby warning, it's probably because you're using JRuby 1.7 in combination with Java objects you've turned into singletons or to which you've added instance variables.

JRuby 1.7 deprecates using instance variables or singletons on Java objects without first calling ```__persistent__ = true``` on the object's Java class. JRuby 1.7 and JRuby 9k will both attempt to make Java objects persistent after the first use of instance vars or singletons, but in 9k we will warn for each case. We recommend using other mechanisms to store per-Java-object data, and you may want to pass ```-Xji.objectProxyCache=false``` to JRuby (or ```-Djruby.ji.objectProxyCache=false``` to Java) to find and fix all such patterns in your code before JRuby 9k.

See the warning at the bottom of this page for known issues with singletons and instance variables on Java objects.

To set a Java class to be persistent, use code similar to the following:

```ruby
java_import java.util.ArrayList
list = ArrayList.new

# These lines will warn if ArrayList is not set persistent yet
class << list; end
list.instance_variable_set(:@foo, 'bar')

ArrayList.__persistent__ = true

# no warning for these lines, but all ArrayList will now have cached proxies
class << list; end
list.instance_variable_set(:@foo, 'bar')
```

This does not cascade down-hierarchy; it must be done for each individual Java class whose instances you expect to use instance vars or singletons with.

For more information, read on.

Java Object Proxy Caching
-------------------------

In order to pass Java objects through Ruby call paths in JRuby, we wrap them in a thin wrapper called the "proxy" or "Java object proxy". These wrappers hold the original Java object and allow it to behave like a Ruby object throughout the system.

In JRuby versions 1.6.x and lower, we also cached these proxies in an attempt to ensure the same object would get the same proxy every time it entered Ruby. This was primarily done to support user-defined instance variables or singleton objects on arbitrary Java objects, since it is not possible to actually modify the structure of the Java object.

However, this caching incurred a heavy cost. Each Java object entering Ruby caused the creation of a weak reference and structural elements in an internal map called the ObjectProxyCache. This greatly increased the overhead of receiving objects from Java calls, and potentially hindered efficient garbage collection due to all those weakrefs.

Deprecating Proxy Caching
-------------------------

As of JRuby 9k, the proxy cache will become opt-in on a per-class basis. If you intend to set instance variables or create singletons from Java objects, you'll need to call JavaClassNameHere.```__persistent__ = true``` on the Java class in Ruby code. This will specify that all that class's objects should have their proxies cached.

We will also attempt to turn on this flag lazily, but with a warning. Because objects may enter Ruby through multiple paths before you set an instance variable or make a singleton, we can't guarantee all existing proxies will reflect those lazily-made changes. We recommend setting ```__persistent__ = true``` on classes before receiving or constructing instances of them.

See the example above for example usage of ```__persistent__ = true```.

JRuby 1.7 will still cache all proxies by default, but will issue a one-time warning if you're using instance variables or singletons on Java objects. We recommend passing ```-Xji.objectProxyCache=false``` to JRuby (or ```-Djruby.ji.objectProxyCache=false``` to Java) to find and fix all uses of instance vars or singletons on Java objects in your code.

Alternatives
------------

The simplest alternative would be to store this attached state in a separate structure, like a Hash (properly mutexed) somewhere in the current JRuby environment. This will allow you to localize the caching perf hit to just those objects where you need it.

You may also wish to reference objects weakly, as we did in ObjectProxyCache. The [weakling](https://rubygems.org/gems/weakling) gem provides a solid, consistent weak-reference implementation as well as a WeakHash implementation.

Warning
-------

Whether you set ```__persistent__``` or not, you *must* keep a reference to the Ruby wrapper object alive in order for it to stay in memory. The caching mechanism JRuby uses allows for either the original Java object or the proxy to be garbage collected if they are no longer referenced, so that the proxy caching does not cause objects to stay alive past their normal lifespan.

Put another way: If you are adding instance variables to or making a singleton object of a Java object, the Ruby object wrapper must stay alive to persist your modifications.