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