<!-- **IN PROGRESS** -->

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

* Warbler has had issues with pre-compiled .rb files due broken IR de-serialization logic, we expect all issues to be fixed and added specs to cover previously failing issues.

### [Issues/Features Resolved in 9.1.0.0](https://github.com/jruby/jruby/issues?q=milestone%3A%22JRuby+9.1.0.0%22+is%3Aclosed)