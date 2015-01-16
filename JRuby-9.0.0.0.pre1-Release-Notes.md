**IN PROGRESS**

These are the release notes for JRuby 9.0.0.0.pre1, the first preview release of JRuby 9000.

Features known to be missing or incomplete:

* Enumerator#feed, #next_values, #prev_values missing
* Kernel#fork
* Kernel#spawn cloexec and chdir support
* ObjectSpace::WeakMap#each and Enumerable inclusion
* ObjectSpace::count_objects
* Process::daemon, exec, getsid, and setproctitle
* RubyVM namespace (specific to MRI)
* Thread#handle_interrupt is not yet fully functional
* POSIX-friendly IO, TTY, and Process logic is not used on Windows

## Truffle

JRuby 9000 includes an in-development version of support for the Truffle language implementation framework and Graal VM from Oracle Labs. In future releases, Truffle will provide an extremely high performance and compatible backend for JRuby. The Truffle backend supports all Ruby language features, but so far only some of the core and standard libraries. It has no support for RubyGems or Rails, does not work on Windows, and is not ready to be tested with applications at this stage. More information on Truffle and Graal can be found in the [JRuby Wiki](https://github.com/jruby/jruby/wiki/Truffle).