# JRuby Command Line Parameters

These JRuby commands are entered in a terminal window or command window, depending on your operating system.

Note: when you run `jruby` from a terminal or command window, you’re running a script that invokes `java` with the proper arguments to start JRuby as specified, for example [`jruby.bash`](https://github.com/jruby/jruby/blob/master/bin/jruby.bash) or [`jruby.bat`](https://github.com/jruby/jruby/blob/master/bin/jruby.bat). Note that some of the below switches apply only to those scripts; they will not work when running JRuby as a JAR file (for example, one created with [Warbler](https://github.com/jruby/warbler)).

```
$ jruby --help
Usage: jruby [switches] [--] [programfile] [arguments]
  -0[octal]         specify record separator (\0, if no argument)
  -a                autosplit mode with -n or -p (splits $_ into $F)
  -c                check syntax only
  -Cdirectory       cd to directory, before executing your script
  -d                set debugging flags (set $DEBUG to true)
  -e 'command'      one line of script. Several -e's allowed. Omit [programfile]
  -Eex[:in]         specify the default external and internal character encodings
  -Fpattern         split() pattern for autosplit (-a)
  -G                load a Bundler Gemspec before executing any user code
  -i[extension]     edit ARGV files in place (make backup if extension supplied)
  -Idirectory       specify $LOAD_PATH directory (may be used more than once)
  -J[java option]   pass an option on to the JVM (e.g. -J-Xmx512m)
                      use --properties to list JRuby properties
                      run 'java -help' for a list of other Java options
  -Kkcode           specifies code-set (e.g. -Ku for Unicode, -Ke for EUC and -Ks
                      for SJIS)
  -l                enable line ending processing
  -n                assume 'while gets(); ... end' loop around your script
  -p                assume loop like -n but print line also like sed
  -rlibrary         require the library, before executing your script
  -s                enable some switch parsing for switches after script name
  -S                look for the script in bin or using PATH environment variable
  -T[level]         turn on tainting checks
  -U                use UTF-8 as default internal encoding
  -v                print version number, then turn on verbose mode
  -w                turn warnings on for your script
  -W[level]         set warning level; 0=silence, 1=medium, 2=verbose (default)
  -x[directory]     strip off text before #!ruby line and perhaps cd to directory
  -X[option]        enable extended option (omit option to list)
  -y                enable parsing debug output
  --copyright       print the copyright
  --debug           sets the execution mode most suitable for debugger
                      functionality
  --jdb             runs JRuby process under JDB
  --properties      List all configuration Java properties
                      (prepend "jruby." when passing directly to Java)
  --sample          run with profiling using the JVM's sampling profiler
  --profile         run with instrumented (timed) profiling, flat format
  --profile.api     activate Ruby profiler API
  --profile.flat    synonym for --profile
  --profile.graph   run with instrumented (timed) profiling, graph format
  --profile.html    run with instrumented (timed) profiling, graph format in HTML
  --profile.json    run with instrumented (timed) profiling, graph format in JSON
  --profile.out     [file]
  --profile.service <ProfilingService implementation classname>
                    output profile data to [file]
  --client          use the non-optimizing "client" JVM
                      (improves startup; default)
  --server          use the optimizing "server" JVM (improves perf)
  --dev             prioritize startup time over long term performance
  --manage          enable remote JMX management and monitoring of the VM
                      and JRuby
  --headless        do not launch a GUI window, no matter what
  --bytecode        show the JVM bytecode produced by compiling specified code
  --version         print the version
  --disable-gems    do not load RubyGems on startup
```