There are many ways to pass options to both JRuby and the underlying JVM. Because it can sometimes be confusing to understand where these options are coming from, we have added in JRuby 9.2.9 the ability to log the JRuby launcher's environment and options processing using the `--environment` flag.

Inspecting the JRuby environment
--------------------------------

JRuby's command-line launcher provides a `--environment` flag you can use to print out information about the JVM, environment variables, and JRuby installation that we use to launch JRuby.

Example output:

```
$ jruby.bash --environment -e 1
JRuby Environment
=================
JRuby executable: /Users/headius/projects/jruby/bin/jruby.bash
JRuby command line options: --environment -e 1
JRUBY_HOME: /Users/headius/projects/jruby
JRUBY_OPTS: 
JAVA_OPTS: 
JAVACMD: /Library/Java/JavaVirtualMachines/jdk-11.0.2+9-hotspot/Contents/Home/bin/java
JAVA_HOME: /Library/Java/JavaVirtualMachines/jdk-11.0.2+9-hotspot/Contents/Home
Adding Java options from: /Users/headius/projects/jruby/bin/.jruby.java_opts
Adding Java options from: /Users/headius/projects/jruby/.jruby.java_opts
Adding module flags from: /Users/headius/projects/jruby/bin/.jruby.module_opts
Java command line: /Library/Java/JavaVirtualMachines/jdk-11.0.2+9-hotspot/Contents/Home/bin/java @/Users/headius/projects/jruby/bin/.jruby.java_opts @/Users/headius/projects/jruby/.jruby.java_opts -Xss2048k @/Users/headius/projects/jruby/bin/.jruby.module_opts -Djffi.boot.library.path=/Users/headius/projects/jruby/lib/jni -Dfile.encoding=UTF-8 -Djava.security.egd=file:/dev/urandom --module-path /Users/headius/projects/jruby/lib/jruby.jar -classpath : -Djruby.home=/Users/headius/projects/jruby -Djruby.lib=/Users/headius/projects/jruby/lib -Djruby.script=jruby -Djruby.shell=/bin/sh org.jruby.Main -e 1
```

This information may help you figure out configuration problems. We recommend including this output when filing a JRuby issue.