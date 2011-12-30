# Tips

[[Troubleshooting Memory Use]] - for when your application seems to use too much memory on JRuby.

[[Improving Startup Time]] - tips and tricks for improving startup time.

## Running JRuby embedded on Solaris
There is [a report](http://bugs.jruby.org/6090) that GC may not work correctly. Set `-Djruby.native.enabled=false` to avoid this problem.