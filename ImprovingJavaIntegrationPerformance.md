Improving Java Integration Performance
======================================

Calling Ruby from Java involves a few challenges for JRuby:

* Objects may need to be coerced to or from their Java equivalents, like Ruby String to Java String
* The target method may have multiple overloads from which to pick
* Java's reflection subsystem may introduce overhead into the calls
* JRuby 1.7 and below will attempt to cache object wrappers, and this caching introduces overhead

This page provides some tips on how to improve the performance of calling Java methods from Ruby.

Avoiding Java Integration altogether
------------------------------------

The "nuclear option" to improve Java Integration performance is to not use it. Calls from Ruby to Ruby have less overhead than calls from Ruby to Java, and if you wrap a Java library in a purpose-built JRuby extension (itself probably written in Java to JRuby's extension API), that will also avoid Java Integration overhead. However you can, by using the tips below, achieve the same level of performance with Java Integration in JRuby 1.7 and higher on Java 7 (where invokedynamic helps a lot).

The rest of this article will focus on improving Java Integration perf, rather than avoiding it.

Pre-coerce values used repeatedly
---------------------------------

If calling methods from Ruby with the same coercible Ruby values, for example the same Ruby String repeatedly, pre-coercing the Ruby String to a Java String will avoid that coercion happening for every call.

```
str = "my string"
obj = StringReceiver.new
obj.receiveString(str)       # must coerce str every time
str_java = str.to_java       # pre-coerce to have a Java String in hand
obj.receiveString(str_java)  # no coercion, value goes straight in
```

Provide signature-specific aliases for overloaded methods
---------------------------------------------------------

When a target Java method has multiple overloads, JRuby must choose among them for every call. JRuby caches matches it has seen, but there's still a lookup process which adds overhead. Also, newer features of JRuby like its use of invokedynamic do not yet optimize overloaded Java calls.

You can use `java_alias` to create signature-specific aliases of a given method, avoiding the lookup overhead and allowing invokedynamic to optimize the calls.

```
# Our Java class, org.jruby.util.ByteList has three overloads for #append.
#
# append(int)
# append(byte[]) <== We want this one
# append(ByteList)

java_import org.jruby.util.ByteList

target = ByteList.new 
foo_bytes = ByteList.plain("foo")  # 'foo' byte[]
target.append(foo_bytes) # must choose from the three signatures every time

class ByteList
  # Add an alias for the byte[] overload
  java_alias :append_bytes, :append, [Java::byte[]]
end

target.append_bytes(foo_bytes) # direct call, no lookup, optimizable
```

Avoid the proxy cache
---------------------

As described on the [[Persistence]] page, JRuby versions 1.7 and earlier attempt to cache the wrappers we put around Java objects. This "persistence" of wrappers has a very high cost, and will be lazy or opt-in by default in JRuby versions after 1.7. By turning off wrapper persistence (via -Xji.objectProxyCache=false), Java calls that send or receive Java objects will not have the additional overhead of caching their wrappers.

Putting it all together
-----------------------

Here's a simple benchmark of appending a byte[] to a ByteList via a Java integration call, and the equivalent logic in the String#<< method. First, the benchmark:

```
java_import org.jruby.util.ByteList

puts "Measure bytelist appends (via Java integration)"
5.times { 
  puts Benchmark.measure { 
    sb = ByteList.new 
    foo = ByteList.plain("foo") 
    1000000.times { 
      sb.append(foo) 
    } 
  }
}

puts "Measure string appends (via normal Ruby)"
5.times { 
  puts Benchmark.measure { 
    str = "" 
    foo = "foo" 
    1000000.times { 
      str << foo 
    } 
  } 
}
```

The results with none of the above tips in play:

```
Measure bytelist appends (via Java integration)
  0.285000   0.000000   0.285000 (  0.285000)
  0.177000   0.000000   0.177000 (  0.177000)
  0.187000   0.000000   0.187000 (  0.187000)
  0.168000   0.000000   0.168000 (  0.168000)
  0.172000   0.000000   0.172000 (  0.172000)
Measure string appends (via normal Ruby)
  0.113000   0.000000   0.113000 (  0.113000)
  0.075000   0.000000   0.075000 (  0.075000)
  0.076000   0.000000   0.076000 (  0.076000)
  0.075000   0.000000   0.075000 (  0.075000)
  0.076000   0.000000   0.076000 (  0.076000)
```

As you can see, the Java call has significantly more overhead, even though the resulting work done is nearly identical in both cases.

Now, we modify the benchmark according to the tips above and run with -Xji.objectProxyCache=false:

```
class ByteList
  java_alias :append_bytes, :append, [Java::byte[]]
end

puts "Measure bytelist appends (via Java integration)"
5.times { 
  puts Benchmark.measure { 
    sb = ByteList.new 
    foo = ByteList.plain("foo") 
    1000000.times { 
      sb.append_bytes(foo) 
    } 
  }
}
```

The resulting numbers show that the Java call now has no overhead compared to the Ruby call. Both calls are direct and optimized well by JRuby and invokedynamic:

```
Measure bytelist appends (via Java integration)
  0.194000   0.000000   0.194000 (  0.193000)
  0.107000   0.000000   0.107000 (  0.107000)
  0.083000   0.000000   0.083000 (  0.083000)
  0.082000   0.000000   0.082000 (  0.082000)
  0.084000   0.000000   0.084000 (  0.084000)
Measure string appends (via normal Ruby)
  0.109000   0.000000   0.109000 (  0.108000)
  0.107000   0.000000   0.107000 (  0.107000)
  0.081000   0.000000   0.081000 (  0.081000)
  0.080000   0.000000   0.080000 (  0.080000)
  0.082000   0.000000   0.082000 (  0.082000)
```

(Note: Minor variation in these numbers is expected; the important detail is that both runs are now roughly equivalent speed)