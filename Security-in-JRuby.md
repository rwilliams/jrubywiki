For now this aggregates some links to security information and a few FAQs.

FAQs
----

* How is JRuby affected by the [hash collision DOS](http://www.ocert.org/advisories/ocert-2011-003.html)

JRuby versions prior to 1.6.5.1 are affected. JRuby 1.6.5.1 includes a patch to randomize the hash value for String, making this much harder to exploit. Thomas Enebo explained the DOS and fix in more detail at [Special JRuby Release 1.6.5.1](http://www.engineyard.com/blog/2011/special-jruby-release-1-6-5-1/).

JRuby 1.7 will also include a method that allows you to get the non-seeded hash, for applications that would like a fast String#hash that is predictable. It uses murmurhash with an initial seed hash of 0.

```
require 'jruby/util'
puts 'foo'.hash, 'foo'.unseeded_hash
```

Run twice:

```
system ~/projects/jruby $ jruby unseeded_hash.rb 
265581630
-1880464523

system ~/projects/jruby $ jruby unseeded_hash.rb 
1788498603
-1880464523
```