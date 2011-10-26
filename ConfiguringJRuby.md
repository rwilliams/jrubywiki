Configuring JRuby
=================

As of version 1.6.5, JRuby now supports a .jrubyrc file in either the current directory or the user's home directory. The format of this file is a series of key/value pairs, with the key being the property name from ```--properties``` and the value being the value you'd pass on the command line for the same property.

For example, to set the default Ruby version mode to 1.9, you'd use the following line:

```
compat.version=1.9
```

A configuration file that sets 1.9 mode, disables C extensions, and generates full backtraces for the EAGAIN errno would look like:

```
compat.version=1.9
cext.enabled=false
errno.backtrace=true
```

Most properties should be documented in the ```jruby --properties``` output, but for a complete listing look at src/org/jruby/RubyInstanceConfig.java.