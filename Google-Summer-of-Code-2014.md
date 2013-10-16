# Wrapper for Matthias Mann's continuations library

The Continuations Library by Matthias Mann provides the basis for lightweight coroutines on the JVM used by the Pulsar and Quasar libraries to achieve feats like 10,000 actors on the JVM.

It would be great if this library could be leveraged from JRuby, either with a proprietary API, or with an implementation of Fibers which is backed by this library.