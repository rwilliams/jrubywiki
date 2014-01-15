This page hosts the ideas for Google Summer of Code 2014! Add your ideas here, improve others, and if you're a student, perhaps something on this list will interest you!

Ideas
=====

### Native coroutine support

Implement some kind of native coroutine support for JRuby. Bonus points for an implementation which is compatible with the Fiber API. One potential approach:

The [Continuations Library](http://www.matthiasmann.de/content/view/24/26/) by Matthias Mann provides the basis for lightweight coroutines on the JVM used by the [Pulsar](https://github.com/puniverse/pulsar) and [Quasar](https://github.com/puniverse/quasar) libraries to achieve feats like [10,000 actors on the JVM](http://blog.paralleluniverse.co/post/64210769930/spaceships2).

It would be great if this library could be leveraged from JRuby, either with a proprietary API, or with an implementation of Fibers which is backed by this library.

### Celluloid "Turbo Mode" for JRuby

[Celluloid](http://celluloid.io) is an actor-based concurrent object framework (somewhat similar to Akka) written in pure Ruby. This means it presently uses Ruby Mutexes and ConditionVariables for synchronization. However, the JVM has many, many other options which could provide better performance. Celluloid provides an [ActorSystem](https://github.com/celluloid/celluloid/blob/master/lib/celluloid/actor_system.rb) abstraction for supporting multiple different platform-specific backends, and we'd love to have one specific to JRuby.

The goal of this project would be to implement a duck type of the `Celluloid::Mailbox` class that leverages native JVM facilities to improve performance. Some options to consider:

* ***[LMAX Disruptor](http://lmax-exchange.github.io/disruptor/)***: Disruptor is a library which supports a number of different patterns for multithreaded execution. The main way LMAX could benefit Celluloid would be providing a way to preallocate and recycle inter-actor messages, storing them in a RingBuffer and providing cache-friendly operation while reducing the allocation rate and thus the demands on the GC. It's unclear if Disruptor's concurrency model could map to Celluloid's well, but it could be used in conjunction with the above data structures specifically for the purposes of leveraging preallocation. [Some work has already been done to implement Celluloid Mailboxes in terms of Disruptor](https://github.com/celluloid/celluloid/issues/342)
* ***[ArrayBlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ArrayBlockingQueue.html)***: These are fast, fixed-sized data structures built atop arrays. Their bounded size might require some semantic changes to Celluloid (see [this ticket for discussion on bounded mailboxes](https://github.com/celluloid/celluloid/pull/153)) but are probably the simplest way to improve performance on Celluloid.
* ***[LinkedTransferQueue](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedTransferQueue.html)***: Introduced in Java 7, LinkedTransferQueue could provide Celluloid's existing unbounded semantics with better performance than Java's previous linked queues. LinkedTransferQueues are a bit complicated and support lots of different modes of operation, so mapping them specifically to Celluloid's semantics might be a bit difficult.
