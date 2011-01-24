Downloading JRuby Source and Building It Yourself
=================================================

If you prefer to download JRuby source and build it yourself, rather than installing a binary as described in [[Getting Started with JRuby|GettingStarted]], you can either download the source files from our web site or get JRuby source code directly from our git repository. 

* To get JRuby source from the web in a tar or zip file, [Download the latest JRuby Source .tar or .zip file](http://jruby.org/download) and extract it to a directory.  The instructions below assume you've extracted the file to the `jruby` directory. 
  * For OSX, Linux, BSD, Solaris, and other UNIX varieties, get the most recent `jruby-src-X.Y.Z.tar.gz` file.
  * If you're on Microsoft Windows, get the most recent `jruby-src-X.Y.Z.zip` file.
* To get JRuby source from our git repository, retrieve the trunk version of the JRuby source. For example, you can use the following terminal window or command window command to save the files to the `jruby` directory:

    git clone git://jruby.org/jruby.git

Build JRuby using the following shell commands ('''Note:''' Ant 1.7 is required to build JRuby version greater than 1.1):

    cd jruby
    ant

Create an up-to-date version of `jruby-complete.jar` with this ant task:

    ant jar-complete

Generate an up-to-date set of the JavaDoc for JRuby located at `docs/api/index.html`:

    ant create-apidocs # (< Jruby 1.3.*)
    ant apidocs #(>= Jruby 1.4.*)

Delete any build and compile artifacts:

    ant clean

Run the JRuby tests:

    ant test

See [[GettingStarted#Did_It_Work|Did It Work?]] for information on using JRuby.