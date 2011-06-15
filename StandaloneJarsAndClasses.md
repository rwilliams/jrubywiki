Often you may want to generate a "standalone" artifact using JRuby plus your own Ruby code. This may be in the form of a "real" Java .class file, or it may be in the form of an executable .jar file.

Standalone Class Files
----------------------

Many Java applications need to have real .class files on disk. Normally, JRuby's compiler does not produce .class files that look like typical Java classes. It is possible, however, to use the `--java` or `--javac` flags to JRuby's compiler to generate normal .class files from simple Ruby class structures, allowing them to be used from Java as though they were normal Java classes. See [[Generating Java Classes|GeneratingJavaClasses]].

Standalone Executable Jar Files
-------------------------------

You may also want to bundle JRuby plus your own Ruby code  into a single .jar file that when run launches your application. Starting with JRuby 1.6, you can do this by changing the `Main-Class` for the jar file to point at `org.jruby.JarBootstrapMain` and adding your own jar-bootstrap.rb to the root of the jar file. This file will be loaded and launched as though it were specified on the command line.

Example:

    ~/projects/jruby $ cp lib/jruby.jar myapp.jar
    
    ~/projects/jruby $ cat jar-bootstrap.rb
    puts "hello"
    
    ~/projects/jruby $ jar ufe myapp.jar org.jruby.JarBootstrapMain jar-bootstrap.rb
    
    ~/projects/jruby $ java -jar myapp.jar
    hello

Here, we've taken the base jruby.jar file, copied it to a new name, modified the "Main-Class" by using the -e flag to 'jar', and added our own jar-bootstrap. You can also embed additional .rb files into the jar, and the root of the jar file will be used as an implicit LOAD_PATH entry.

See also the [Warbler](https://github.com/nicksieger/warbler) utility for packaging up a command-line or web application as a standalone .jar or .war, including gem dependencies. 