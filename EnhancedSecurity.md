$SAFE shows it's age.  It is an acknowledged stolen feature from Perl and what does '$SAFE = 3' mean anyways?

Let's give security a facelift.  Let's make a new variable with some builtin feature and extensibility as we realize new aspects of security we want to monitor.

    require 'security'

    $SECURITY[:io] = :warn

This would tell Ruby that the IO subsystem should give warnings on any use of IO, but not actually stop working.  Useful for auditing.  If you are on a locked down system:

    $SECURITY[:io] = :error

This will throw a SecurityError on any use of IO.  But let's say you want your own policy:

    $SECURITY[:io] = MyIOPolicy.new

Ok hand-waving is over and obviously this is sparse on details.  Here are the main ideas:

1. $SECURITY is multi-dimensional on facilities it can enforce and therefore extensible over time.  :io was given as an example, but any named facility can be provided.  This also is much more intuitive than 1-5.
1. Policy enforcement is a value.  Builtin values can expand over time like :warn and :error, but a user-defined policy can also exist.
1. Each facility defined should have a set of conformance specs.  Lack of coverage in reality should lead to a new set of specs to verify conformance.  This develops confidence in the facility.  Conformance only ensures core conformance.  It does not help for extensions to the implementation.

A few concerns about the proposal:

1. This is prone to the same brittleness where missing a place where a facility is used is akin to busting a hole in your runtime.  Ain't security a bitch.  [Note: See above about spec converage for proof of conformance.  Not perfect, but good way of ensuring eventual convergence]
1. People may not realize that user-defined policy will have some serious consequences on overall performance.

Questions:

1. What values should actually be passed into a policy?
1. Should the interface for policies be uniform or match what the policy represents?
1. $SECURITY implies it is everywhere (unfortunately MRI violates meaning of $ regularly but it still feels wrong).  We would like something which implies per-thread possibilities while also making it clear when you want global policies.  We mentioned Thread.security[:io] where this works for current thread and all children.
