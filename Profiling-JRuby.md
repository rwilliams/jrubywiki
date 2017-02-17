Using the built-in profiler
===========================

Turning on the built in profiler
--------------------------------

JRuby has a built-in profiler that you can use in several ways.  The simplest way is to just add the `--profile` flag on the commandline, or in `JRUBY_OPTS`.  This outputs a flat profiler and is available as of JRuby 1.5.

In JRuby 1.6+, you can expand the above to --profile.flat for the (default) flat profile, `--profile.graph` for a graph profile, or `--profile.api` to turn on the profiling API, which allows you to selectively turn the profiler on and off directly from your Ruby code.

Outputting to a file (JRuby 1.7.4+)
-----------------------------------

You can output the results of profiling to a file by specifying ```--profile.out <file>``` as a flag to JRuby. Normally, profiling output is sent to stderr, which can cause it to be intermingled with program output.

Profiling an entire application
-------------------------------

If you want to profile all of your application's code, simply using `--profile`, `--profile.flat`, `--profile.graph` or `--profile.html` will do a great job for you.  This is generally best if most of the code that executes in your application is yours, or if you want to take a look at what is consuming most of the time in your application.  Once you have isolated an area that you think could use improvement, either through results in load or performance tests, or through broad profiling as above, you'll likely want to get more specific.


Profiling specific code in an application
-----------------------------------------

To get more specific, you're going to want to wrap the specific section of code you'd like to profile in a block that you pass to `JRuby::Profiler.profile`.  Here's an example, taken from Daniel Lucraft's blog: http://danlucraft.com/blog/2011/03/built-in-profiler-in-jruby-1.6/

    require 'jruby/profiler'
    
    profile_data = JRuby::Profiler.profile do
      # code to be profiled:
      [*50..1000].collect {|val| val = val + 10}
    end

Next, you'll want to use one of the profile printers to format the data in a way that you can easily consume.  The options for this are:

* GraphProfilePrinter
* FlatProfilePrinter

If you wanted to replicate the behavior of the `--profile.graph`, but focused only on the block you wrapped above, you could output it in this manner, printing to STDOUT:
profile_printer = JRuby::Profiler::HtmlProfilePrinter.new(profile_data)

    ps = java.io.PrintStream.new(STDOUT.to_outputstream)
    printer.printHeader(ps)
    printer.printProfile(ps)
    printer.printFooter(ps)

You could also write it to a file, or any IO stream.  One example of this can be found at: https://gist.github.com/872355

**Don't forget** to run JRuby with `--profile.api` to turn on the profiling API, otherwise no profiling data is recorded.