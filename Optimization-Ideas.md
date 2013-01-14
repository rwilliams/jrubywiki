Collection of rough ideas jruby-internal optimizations. Flashes of insight welcome.

## Thread.kill/Thread.raise

1. method entry and loop backedges contain thread mailbox read. switchpointed to a NOP by default
1. on the first mail the switchpoint is flipped
1. actual site were a mail is caught is recorded. this site will permanently perform polling from now on. (possibly after some walltime delay)
1. remaining callsites can be reoptimized with a switchpoint
1. Next thread.kill will optimistically wait under the assumption that kills are localized to busy loop and that we already installed permanent polling at the right site. After a timeout return to step 2.