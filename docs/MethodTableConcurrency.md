Method Table Concurrency
========================

JRuby does not make hard guarantees about the visibility of method table changes across threads. This means that under heavy concurrency, threads making modifications to a class or updating a given call site may not
see their changes reflected immediately in other call sites.

In practice, this does not actually cause problems for a few reasons.

Call Site Invalidation
----------------------

There are two ways call sites are invalidated in JRuby.

### Type Invalidation

If a call site has cached a method from one type, and a future call sees a different type, then the cache must be invalidated.

Pre-invokedynamic, type invalidation is done by comparing a cached serial number with the incoming class's serial number. Because call sites are always *actively* guarded in this way, one thread's update to a call site does not need to propagate immediately; the other thread will simply see the bad serial number and perform its own invalidation.

With invokedynamic, the type comparision is an exact object identity comparison between the cached class and the incoming class. Again, this is an *active* guard, so failure to propagate from one thread to another is non-fatal.

### Modification Invalidation

Because classes can be modified at any time, we must also guard against a cached type being modified.

Pre-invokedynamic, modification invalidation piggy-backs on the serial number. Class modifications cascade down-hierarchy, updating the serial numbers of all child classes or including hierarchies. The one serial number check covers both type identity and modification.

With invokedynamic, modification checks are performed using a SwitchPoint per class. The SwitchPoint is used in addition to the type identity comparison, and ultimately a successful cache will optimize down to *just* the type identity check. Because SwitchPoint is explicitly propagated to all threads, concurrent modifications of a class will permanently invalidate any call sites in any threads that have cached methods from that class. Again, though we don't immediately update the call sites across threads, the SwitchPoint guards against seeing a bad cache.