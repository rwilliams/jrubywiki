Classpath and Load Path
=======================

Prior to JRuby 1.1, Java's Classpath (CLASSPATH) and Ruby's Load Path were largely separate. This document describes the unification of the two paths in JRuby 1.1.

Load Path
---------

JRuby supports the concept of a "load path" as per Ruby 1.8. The load path contains filesystem directories to be used as the "root" when searching for scripts to load or execute.

Script files contained within the load path can be loaded using the `require` or `load` kernel methods. The differences between these two mechanisms are discussed in more depth on the [http://www.headius.com/rubyspec RubySpec Wiki].

Load path is a lazier, more implicit form of pathing than Java's classpath, since libraries on the load path must still be explicitly loaded to become available.

Classpath
---------

Java applications have their own path logic used for locating `.class` files, called the ''classpath''. The classpath is typically set up through the CLASSPATH environment variable or passed to the `java` command using `-cp` or `-classpath` with a delimited list of filesystem locations.

Classpath entries typically must contain one of two things:

* The root from which a directory structure containing class files can be searched.
* A JAR file containing that same structure.

Classpath is a more explicit form of pathing than load path, since everything in the classpath can be referenced from compiled code. No load step is needed, though the JVM will perform a load-on-demand as class dependencies require.

Classpath and Load Path Unification
-----------------------------------

Because the operation of Java's classpath and Ruby's load path are so similar, especially under JRuby, they are unified in JRuby 1.1. This results in a number of unified capabilities:

* Everything in the Ruby load path is considered to be a classpath entry, so `.class` files under load path hierarchies are automatically available to be referenced from code.
* Everything in the Java classpath is considered to be a load path entry, so `.rb` scripts, for example, contained in JAR files, are loadable.
* Structures specific to the Java classpath (such as JAR files and remote URLs) can be specified as load path entries. This allows explicitly adding JAR files and remote servers locations to the JRuby load path, as though they were local paths.

Largely, this unification is intended to make it simpler to work with hybrid applications, where some code is in Ruby and some is in Java, and especially cases where Ruby code is being pre-compiled to Java bytecode and straddles the two worlds.
