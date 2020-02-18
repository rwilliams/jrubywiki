Because JRuby lives on the JVM, there's a wide array of both Ruby and JVM-level tools for investigating poor performance on your JRuby application. This page attempts to lay out the typical sequence we have users follow.

1. Use the JVM sampling profiler with `--sample`.

   JRuby provides a convenience flag `--sample` that enables a JVM-level sampling profiler. Because it is sampled, the results are not super accurate, but serious problems often show up in this output. Each thread will get its own sampling profile, dumped to the console when the thread ends. See [[Output of JVM Sampling Profiler]] for an example.

   Alternatively, you can enable the "hprof" sampling profiler (shipped with all OpenJDK builds and possibly other JVMs as well) by passing `-J-Xrunhprof:cpu=samples` to JRuby. This version will build a single report at exit aggregating all threads' samples, and output the result to a file named `java.hprof.txt` in the current directory. The samples will be at the bottom of the file, and the top of the file will contain stack traces for the sampled results.

2. Use JRuby's [built-in Ruby profiler](https://github.com/jruby/jruby/wiki/Profiling-jruby) with `--profile`.

   JRuby ships with a built-in profiler that works at a Ruby level. Enable it with --profile. The output will be a listing of all methods called with call counts and timings for each. This works well to find a single Ruby method that's consuming too much time.

3. Run an allocation profile with `-J-Xrunhprof:depth=0`.

   The "hprof" profiler also has a mode where it can report information about object allocation. If you pass `-J-Xrunhprof:depth=0` to JRuby, this will run the JVM with instrumentation to track all allocation counts. The `depth=0` part tells hprof not to also split up those counts by nearby backtrace frames.

   Once you have a good listing of the top objects being allocated, you may bump up the `depth` flag to see where in JRuby and your application the allocations are happening.

4. Use a larger tool for profiling, like VisualVM or YourKit.

   Graphical tools like VisualVM and YourKit have modes for sample profiles, instrumented profiles, allocation profiles, and heap dump analysis. A complete treatment of these tools is beyond the scope of this page, but they usually have good tutorials and fairly straightforward interfaces. The results you see here should be similar to the results from the above flags, but perhaps with lower application overhead and a higher level of visibility into your application.