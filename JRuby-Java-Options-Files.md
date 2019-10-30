JRuby allows passing JVM options at the command line using the `-J<javaoption>` flag. Because these options may be quite verbose, may differ between applications and environments, and may encode important configuration details for a given app, we now support (in JRuby 9.2.9 and higher) a `.jruby.java_opts` file.

Command-line Argument Files
------------------------------

Java 9 [introduced support for command-line argument files](https://docs.oracle.com/javase/9/tools/java.htm#JSWOR-GUID-4856361B-8BFD-4964-AE84-121F5F6CF111), which allows you to aggregate JVM options in a file and include those options on the command line with the syntax `@<filename>` rather than passing them directly. JRuby builds on this feature and supports it on Java 8 via our `.jruby.java_opts` files.

.jruby.java_opts
---------------

The format of `.jruby.java_opts` is as documented in the Java documentation above, with options separated by whitespace (spaces, tabs, newlines, etc) or enclosed in quotes to preserve spaces.

On startup, JRuby will look for `.jruby.java_opts` files in the following locations:

1. In the installed JRuby's `bin/.jruby.java_opts` (empty by default).
2. In the current user's home directory at `~/.jruby.java_opts`.
3. In the current working directory at `./.jruby.java_opts`.

Options passed through JRuby to the JVM via different mechanisms will override each other. Given options are processed in the following order, with later options overriding earlier ones:

1. Any `.jruby.java_opts` files will be processed in the order shown above.
2. The `JAVA_OPTS` environment variable will be processed.
3. Any flags passed at JRuby's command line using `-J<option>` will be processed last.

An example file, used by JRuby's `--dev` flag, can be found in the JRuby install under `bin/.dev_mode.java_opts`.

In order to help users understand where options are coming from, and to inspect the eventual set of JRuby and Java options being used, we added in 9.2.10 the `--environment` flag. See [[Environment Flag]] for more information.

Notes
------

* Argument files are only directly supported by the `java` launcher in Java 9 and higher, but JRuby also supports using `.jruby.java_opts` on Java 8. Because we do this by expanding the file's contents directly into our final `java` command, it's possible for this argument list to exceed some shells' character limit.
* There's no way at present to separate `.jruby.java_opts` files by Java version or JVM implementation, mostly due to the difficulty of detecting what JVM you are using before startup. We are exploring other options here.
* JRuby also support passing JRuby options via `.jrubyrc` files from the user's home or from the current directory. See [Configuring JRuby](https://github.com/jruby/jruby/wiki/ConfiguringJRuby) for more information on `.jrubyrc` files.
* A separate options file, in `bin/.jruby.module_opts`, is used when running on a Java Module-enabled JVM. By default it opens packages that JRuby needs for Ruby compatibility.