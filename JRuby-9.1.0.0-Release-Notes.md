**IN PROGRESS**

JRuby 9000 is the new major version of JRuby, representing years of effort and large-scale reboots of several JRuby subsystems.

Major features of JRuby 9000:

* **Ruby 2.3 compatibility**, minus features listed below https://github.com/jruby/jruby/issues/3479
* A new optimizing runtime based on a traditional compiler design
* New POSIX-friendly IO and Process
* Fully ported encoding/transcoding logic from MRI

## JRuby 9.1.0.0

<!-- Ruby 2.2/2.3 features yet to be implemented: -->

### Notable changes since 9.0.5.0

* removing default `-Xmx` (500MB) setting from jruby.bash. Most users that get JRuby will end up using the bash script. https://github.com/jruby/jruby/issues/3739

* `Thread.new` and others spawning threads (`Fiber`, `Enumerator`) should be a bit faster to start, most importantly they no longer possibly slow-down due low entropy sources on *nix systems due a Java `SecureRandom` instantiation on JRuby's `ThreadContext`. https://github.com/jruby/jruby/pull/3723

#### Java Integration

* Java stack-trace filtering was tuned to not exclude everything under the `org.jruby` package prefix, we still avoid the additional noise from the stack-trace but no longer filter potentially unknown packages (e.g. `org.jruby.rack`) or extension stacks (`org.jruby.ext`). This change also affects `backtrace` information on the Ruby side which might now include more *.java* parts.

* the internal `JavaPackageModuleTemplate` has been refactored into a standalone module-like class, all package stubs `org.kares.jruby` now return a `JavaPackage` instance (which is a `Module`)

* names `throw` and `catch` are no longer allowed as package names using the method format (org.jruby.catch will now dispatch to `Kernel.catch`) - those are not valid Java package names anyways

* `singleton_class` (as well as hook methods such as `singleton_method_added`, `singleton_method_removed`) is no longer allowed as a package-name using the method ('.') format (`org.jruby.singleton_class` will return the Ruby meta-class)

* `pkg.to_s` now returns package-name opposed to "module name" e.g. `java.lang.to_s == 'java.lang'` (note that `inspect` still behaves the same as before: `java.lang.inspect == 'Java::JavaLang'`)

* improved `java.util.Map` equality (`==`, `eql?`) to other map/hash instances - previously wasn't 100% correct

* `java.util.Map` proxies now support all of `Hash` methods (including new ones such as `dig`, `fetch_values` and comparison operators e.g. `<=`)

* Java arrays now handle value equality `==` with Ruby arrays (use `eql?` if types need to match as well). this also affects `===` with native arrays.

* Java's exception hierarchy `java.lang.Throwable` no longer provides a `to_str` method, we feel like this unintentionally slipped probably due misunderstanding the difference between Ruby's `to_s` and `to_str`

* `java.util.Collection` instances are no longer `to_ary` (Array-like) convertible, please note that `to_a` still exists and (for compatibility) `java.util.List` provides `to_ary` as well.

* Warbler has had issues with pre-compiled *.rb* files due broken IR de-serialization logic, we expect all issues to be fixed and added specs to cover previously failing issues.

* **jrubyc** `--jdk5` and `-5` switches were removed (Java 5 has not been supported for a while)

* **jrubyc** `--dir` option now handles absolute paths correctly with the `--target` option

### [Issues/Features Resolved in 9.1.0.0](https://github.com/jruby/jruby/issues?q=milestone%3A%22JRuby+9.1.0.0%22+is%3Aclosed)