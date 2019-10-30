Configuring JRuby
=================

JRuby options and .jrubyrc
--------------------------

JRuby (since 1.6.5) supports a *.jrubyrc* file in either the current directory or the user's home directory. The format of this file is a series of key/value pairs, with the key being the property name from ```--properties``` and the value being the value you'd pass on the command line for the same property.

For more details on JRuby-specific properties and the `.jrubyrc` file, see [[JRuby Options]].

JVM options and .jruby.java_opts
--------------------------------

JRuby allows passing JVM options using the `-J<java option>` flag, for example `-J-Xmx2G` to set the max JVM heap to 2GB on OpenJDK. JVM options with spaces should be specified as separate `-J` flags, as in `-J--add-modules -Jmodule/package=readingmodule` for the Java 9+ `--add-modules` flag.

In order to make these flags easier to manage, JRuby 9.2.10 and higher ship with support for `.jruby.java_opts` files. These files allow you to capture global or application-local JVM options in a single file that is automatically picked up by our launcher.

For more information on passing JVM options using `.jruby.java_opts` files, see [[JRuby Java Options Files]].