# Tips

[[Troubleshooting Memory Use]] - for when your application seems to use too much memory on JRuby.

[[Improving Startup Time]] - tips and tricks for improving startup time.

## Running JRuby embedded on Solaris
There is [a report](http://bugs.jruby.org/6090) that GC may not work correctly. Set `-Djruby.native.enabled=false` to avoid this problem.

## Using C extension support with Warbler/wars/Tomcat

At the time of this writing we don't ship `libjruby-cext.so` in the jruby-jars gem. (That might be grounds for filing a bug/improvement ticket; please update this page if you decide to file one.)

To work around that fact, you'll need to do the following:

1. Locate `libjruby-cext.so` and `libjffi-1.0.so` out of the [JRuby binary distribution's][dist] `lib/native/[platform]` directory. Be sure to choose the right directory for your platform.
2. Copy them somewhere on the system (e.g., `/opt/jruby/lib`), and either modify the places where `ld.so` looks for libraries to add that path or set LD_LIBRARY_PATH (or equivalent) to use that path before launching the JVM process that uses JRuby (Tomcat/application server etc.).

[dist]: http://ci.jruby.org/job/jruby-dist-release/ws/lib/native/