This page covers problems you may have getting JRuby to load its own native libraries or libraries you have supplied to it.

JRuby uses an FFI subsystem (built atop the [jnr](https://github.com/jnr) family of libraries to provide both Ruby's FFI API and to bind native functions we use from Java. Either of these modes can run into issues if there's special configuration required for Java Native Interface (Java's native library API) libraries to load.

First Steps
===========

The first line of attack is to pass `-Xnative.verbose=true` to JRuby itself so it will log errors that occur during startup of our FFI subsystem.

Setting Library Paths
=====================

If errors indicate that our own FFI library (`libjffi` or `jffi.dll`) cannot load, you may be able to force the JVM to look in a different location for the library. It should be provided in our `lib/jni` directory, so you may try passing `-Djava.library.path=<path to lib/jni>`.

If a library you are trying to load with Ruby FFI does not load, or if a library other than libjffi can't load, you may need to configure the equivalent of `LD_LIBRARY_PATH` on your platform, keeping in mind that some OSes may have different paths for 32 versus 64-bit code.

Other Links
===========

Here's some other articles by folks who have resolved native library issues under JRuby.

[[JRuby on Alpine Linux]]