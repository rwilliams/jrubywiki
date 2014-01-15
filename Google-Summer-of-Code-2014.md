This page hosts the ideas for Google Summer of Code 2014! Add your ideas here, improve others, and if you're a student, perhaps something on this list will interest you!

Ideas
=====

### Wrapper for Matthias Mann's continuations library

The [Continuations Library](http://www.matthiasmann.de/content/view/24/26/) by Matthias Mann provides the basis for lightweight coroutines on the JVM used by the [Pulsar](https://github.com/puniverse/pulsar) and [Quasar](https://github.com/puniverse/quasar) libraries to achieve feats like [10,000 actors on the JVM](http://blog.paralleluniverse.co/post/64210769930/spaceships2).

It would be great if this library could be leveraged from JRuby, either with a proprietary API, or with an implementation of Fibers which is backed by this library.