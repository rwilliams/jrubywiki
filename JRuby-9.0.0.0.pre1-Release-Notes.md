**IN PROGRESS**

JRuby 9000 is the new version of JRuby, representing years of effort and large-scale reboots of several JRuby subsystems.

Major features of JRuby 9000:

* Ruby 2.2 compatibility, minus features listed below
* A new optimizing runtime based on a traditional 3-address compiler design.
* New POSIX-friendly IO and Process
* Fully ported encoding/transcoding logic from MRI

## Preview Status

This is a preview release, and we know there's still work to do. We are releasing now to get user feedback on Ruby 2.2 functionality and overall stability. 

Ruby 2.2 features yet to be implemented:

* :"foo" (may be done by pre -- need issue number if not)
* Refinements (may be partially landed for pre definitely include issue)
* Enumerator#feed, #next_values, #peek_values missing
* Kernel#spawn cloexec support
* ObjectSpace::WeakMap#each and Enumerable inclusion
* ObjectSpace::count_objects
* RubyVM namespace (specific to MRI)
* Thread#handle_interrupt is not yet fully functional
* POSIX-friendly IO, TTY, and Process logic is not used on Windows
* Startup time is a bit slower.
* Memory usage is higher.
* Straight-line performance is a little bit slower.

We also have additional work to do on the new runtime:

* Startup time is a bit slower.
* Memory usage is higher.
* Straight-line performance is a little bit slower.

The new runtime gathers more information about Ruby code and performs more analysis and optimization than our old runtime. We will do our best to bring these on par with 1.7.x (or better) before the final release of JRuby 9000.

## Truffle

JRuby 9000 includes an in-development version of support for the Truffle language implementation framework and Graal VM from Oracle Labs. In future releases, Truffle will provide an extremely high performance and compatible backend for JRuby. The Truffle backend supports all Ruby language features, but so far only some of the core and standard libraries. It has no support for RubyGems or Rails, does not work on Windows, and is not ready to be tested with applications at this stage. More information on Truffle and Graal can be found in the [JRuby Wiki](https://github.com/jruby/jruby/wiki/Truffle).