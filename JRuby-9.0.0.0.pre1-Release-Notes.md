**IN PROGRESS**

Intro section recognizing major features:
- MRI 2.2 support
- New runtime (IR + interpreter + JIT + potential of new design)
- New IO + full ported transcoding

Section acknowledging that:
* startup is a bit slower
* memory usage is higher
* eventual performance is a little bit slower
With some language specifying our plans to iron these out by final release.

Section on getting feedback to make sure final is solid.

These are the release notes for JRuby 9.0.0.0.pre1, the first preview release of JRuby 9000.

Features known to be missing or incomplete (we should remove things we can never support like fork -- This list should reflect work to be done):

* :"foo" (may be done by pre -- need issue number if not)
* Refinements (may be partially landed for pre definitely include issue)
* jirb_swing has no output (need issue number)
* Enumerator#feed, #next_values, #peek_values missing
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