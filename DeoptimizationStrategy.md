# Deoptimization

This explains the plans for our deoptimization strategy in IR.

## Full Build (IR0)

When we compile (JIT) we make a full build also referred to as IR0.  This generates IR instructions and then runs a set of conservative compiler passes.  We know this is always a basic and safe level of optimization we can return to if we try a speculative optimization and that optimization fails.

## Speculative Optimizations

  We are developing a profiler which will keep track of how code is called.  Primarily, for now, this will be to look for callsites which are monomorphic.  When we discover a callsite WE THINK is monomorphic we will run speculative optimization(s) on it:

  - inline the call (and it's block if it is provided to the call)
  - specialize on numeric type (long, double from Ruby numeric types)

  These optimizations might fail and we need to detect that.  When we see an assumption fail we will fall back to IR0 at the same semantic point that we are in with the optimized version of the code.  For example:

```text
JIT           Full Build (IR0)
optimized     unoptimized
instr1        instra
instr2        instrb
...           ...
instrn   ->   instrm
...           ...
instrz        instrz
```

So, at instrn we fail a guard we know at that point we need to resume at
instrm.  In order to do this we need to keep track of a few things:

  - which ipc do fall back to in IR0
  - all temp variable state
  - whether we eliminated something which needs to be restored (like dynscope)

## On Guards and Dumping State

Let's say we inline a method.  We add a guard instr which will contain the IPC of where the original callsite was.

The guard itself will throw an exception when it fails and in the JITd method it would look something like:

```java
try {
  guard(ipc);
catch (DeoptimizationException e) {
  e.getIPC();
  // dump state...
}
```

Dumping State is actually just dump all the jvm locals associated with IR temporary variables.  We will maintain a list of locals to temps in IR.  The one wrinkle to this try/catch is that if we have any ensure logic which is supposed to happen because the method is exiting...we need to not call that since we are transferring control to another version of this method.

## Temporary Variable Map

Super important IR design assumption: We will never reuse temps so that we can always remap them back when we deopt.  So regardless of how many passes we apply we will always end up being able to remap back to originals.

Another aspect of this map the JVM locals will know which temps existed after the last pass was run.  It will not know the values for any temps which might have been eliminated by a compiler pass.  Because of this we need to maintain a list of all temps and their deleted values.

## Restoring extra state

DynamicScope can get eliminated.  In this case we will need to restore the scope and know which temp variables were which LocalVariables.  Fortunately, IR creates special temps for this which allows us to fish out the var it was used for.

## Immediate Issues vs Longer Issues

We need to write:

 - guard instr
 - catch deopt and contend with ensure/finally
 - jvm local dumper
 - jvm to local temp map

We may need sooner than later:

 - deleted temp info

And later still:

 - restore dynscope

At this point, we only remove dynscope during a conservative pass and some of our passes cannot run on scopes repeatedly (although this is not desired behavior).  It is possible we delete instr somehow in our CFGInliner and in our Unboxing pass.  If so then we need to record deleted temps.
