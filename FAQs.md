Sections
========
* <a href="#execution_environments">Execution Environments</a>
* <a href="#general">General</a>
* <a href="#running_rails">Running Rails</a>
* <a href="#calling_into_java">Calling Into Java</a>
* <a href="#java_and_jruby_interop">Java and JRuby Interoperation</a>
* <a href="#troubleshooting">Troubleshooting</a>
* <a href="#networking">Networking</a>

<a name="execution_environments"/>
Execution Environments
======================

Can I run JRuby on Java Micro Edition?
--------------------------------------

Somewhat.  See [[JRuby on JME|JRubyOnJavaMicroEdition]].

How can I run JRuby using Java Web Start?
-----------------------------------------

Here is a page dedicated to answering this question: [[JRuby on Web Start|JRubyOnWebStart]]

How can I make my Ruby scripts execute with JRuby on unix systems like other scripts?
-------------------------------------------------------------------------------------

Include a "hash bang" line on the top of your jruby script like this:
```bash
    #!/usr/bin/env jruby
```

That will use the system wide installed jruby interpreter to run your script.

If you would like to use a local copy of JRuby to run your script, give the relative or absolute path to the jruby program:

```bash
    #!/usr/bin/env jruby-1.6.8/bin/jruby
```

To run the script directly, you need to set the "x" flag on it:

```bash
    chmod +x your_script.rb
```

You can now start your script with 

```bash
    ./your_script.rb
```

You can also try the `jruby-launcher gem`, which installs a native `jruby` program.

```bash
    jruby -S gem install jruby-launcher
```

Now you can place the full path to JRuby in the shebang line, along with parameters if you wish:

```bash
    #!/usr/local/jruby/bin/jruby -w
```

How can I increase the heap/memory size when launching a sub-JRuby?
-------------------------------------------------------------------

You can try passing the usual `-J-Xmx<size>` to the sub-JRuby. But beware:

```bash
    $ jruby -e '%x{jruby -J-Xmx256m -e true}'
    warning: -J-Xmx256m argument ignored (launched in same VM?)
```

Historically, JRuby has been very slow to start up, so we have tried to recognize some commands that look like they may be launching more Ruby programs and we start those in the same JVM. In that case, you actually can't adjust the heap.

We have a special system property that controls this behavior VM-wide (pass on the command-line as `-J-Djruby.launch.inproc=false`):

```bash
    jruby.launch.inproc=true|false
    #Set in-process launching of e.g. system('ruby ...'). Default is true
```

And you can also control this from Ruby code for the current JRuby runtime:

```ruby
    require 'jruby'
    JRuby.runtime.instance_config.run_ruby_in_process = false
```

With the `inproc` setting disabled, you can now pass `-J` arguments to the sub-JRuby as it will be launching a new JVM.

In the future, as JRuby and JVM startup performance increases, we may flip the default in-process launching behavior to `false` to match most peoples' expectations. See also [this thread on the JRuby mailing list](http://www.mail-archive.com/dev@jruby.codehaus.org/msg02861.html) for a discussion of `jruby.launch.inproc`.

Why does my application use more memory under JRuby than under <your impl of choice here>?
------------------------------------------------------------------------------------------

See [[Troubleshooting Memory Use]].

Why do I get "Unrecognized option...JVM creation failed" when passing -J flags like -splash?
--------------------------------------------------------------------------------------------

On Windows, the native `jruby.exe` launcher usually loads the JVM's DLL directly and spins up the JVM in
the same process, rather than launching an external process using the `java.exe` command. However, `-splash` and a few other options are only supported by the `java.exe` command and are unrecognized by the DLL.

Because we have no way of knowing what options are or are not supported by the DLL, we provide a flag for the
`jruby.exe` command to force using `java.exe` and spinning an external process: `-Xfork-java`. Passing this flag to the `jruby` command line will allow those other `java.exe`-only flags to work correctly.

<a name="general"/>
General
=======

Where can I find more information about the Ruby language?
----------------------------------------------------------

[The Ruby Language Home Page](http://www.ruby-lang.org) has links to a wide range of resources for learning about Ruby, downloading releases of the C implementation of Ruby, documentation on Ruby, and community outlets for talking about Ruby.

[The Pragmatic Programmers](http://www.pragmaticprogrammer.com/) are the publishers of the de facto standard Ruby text, "Programming Ruby". It is considered a must-have manual to the Ruby language and libraries.

Almost everything you learn about the Ruby language is directly applicable to working with JRuby. JRuby aims to be a drop-in replacement for the C implementation of Ruby.

What incompatibilities does JRuby have with the C implementation of Ruby? 
--------------------------------------------------------------------------

See [[Differences Between MRI and JRuby|DifferencesBetweenMriAndJruby]].

Does JRuby support Ruby 1.9.x features?
---------------------------------------

Yes. JRuby officially started supporting 1.9.2 features as of the 1.6.x line of releases, though it
is not default in those versions. You must specify the `--1.9` flag at the command line or in a *JRUBY_OPTS*
environment variable, or add "compat.version=1.9" to *.jrubyrc* (on 1.6.5 or higher).

Ruby 1.9 mode is default in JRuby 1.7.0 and later, 1.8.7 compatibility can still be used with the `--1.8` flag.

How do I run JRuby without the command-line scripts?
----------------------------------------------------

Frequently users want to run JRuby from NetBeans, from Eclipse, launched from an existing Java application, or in other ways that don't use the command-line scripts. In addition to having `jruby.jar` and its dependency jars available in CLASSPATH, you must also have the following system property set:

```bash
    -Djruby.home=<path-to-root-of-JRuby-installation>
```

You might also want to ensure the maximum memory is set higher than the JVM default. JRuby's own command-line scripts use a maximum of 256MB.

What if I want to use JRuby alongside C Ruby?  How do I keep from getting confused?
-----------------------------------------------------------------------------------

You have two options:

* Always invoke JRuby with e.g., `jruby -S gem`.
* Put a handy bash snippet like this in your .bashrc to create `j` aliases to all the available commands (`rails` becomes `jrails`, `rake` becomes `jrake`, etc.)

```
    for f in $JRUBY_HOME/bin/*; do
      f=$(basename $f)
      case $f in jruby*|jirb*|*.bat|*.rb|_*) continue ;; esac
      eval "alias j$f='jruby -S $f'"
    done
```

Why can't JRuby find my installed gems?
---------------------------------------

JRuby can only see the gems installed with the gem command shipped with JRuby. The gems installed this way will be stored under `$JRUBY_HOME/lib/ruby`. If you try to run Rails and get the error, "Cannot find gem for Rails ~>1.2.2.0:" (or whatever version you are using), the problem is probably that you haven't installed the Rails-gem and its dependencies with the JRuby gem-command.

When I implement a Java interface and provide my own initialize method, I can't construct it anymore.
-----------------------------------------------------------------------------------------------------

In JRuby 0.9.0, any class implementing a Java interface must explicitly call `super` in their initializers to set up the interface proxy. Add a `super` call to your implementation's `initialize` method and it should work.

How do I call JRuby from my existing Java program?
--------------------------------------------------

If you don't want to launch JRuby as a separate process, we recommend you use the [Bean Scripting Framework](http://jakarta.apache.org/bsf/) (BSF) or [Java 6's Scripting Support](https://scripting.dev.java.net/) (JSR223). We do not recommend calling directly into the JRuby runtime, since that code is subject to change.

 Where can I find the javadoc? 
-------------------------------


How do I get a Ruby backtrace when calling JRuby code from Java?
----------------------------------------------------------------

You need to do something along the lines of:

```java
    try {
      rubyCode.doSomething();
    } catch (RaiseException e) {
      e.getException().printBacktrace(System.err);
    }
```

Note that Java 6's scripting via the BSF libraries might not preserve stack traces, and also launches more slowly.  It's often preferable (as of February 2007) to use JRuby's own integration.

Why do I get a `java.lang.NoSuchMethodError: org.objectweb.asm.ClassWriter.visit()` error when running JRuby 1.X with a custom classpath?
----------------------------------------------------------------------------------------------------------------------------------------------------

The problem stems from an ASM jar conflict. It was found by setting a custom classpath, which includes a version of Spring that has a conflicting ASM jar, and then invoking JRuby. The easiest workaround is to build `jruby-complete` as a single jar and edit the JRuby config file (not sure about Windows, but it should be similar) to source only the `jruby-complete` jar. You will need to add ALL other jars, which I  believe includes your jdbc driver as well.

```bash
    svn co http://svn.codehaus.org/jruby/tags/jruby-1_0/
    cd jruby-1_0
    ant jar-complete
    vi bin/jruby (see diff for edit)
```
```diff
    #diff bin/jruby bin/jruby.orig 
    #89c89
    #<     for j in "$JRUBY_HOME"/lib/jruby-complete.jar; do
    #---
    #>     for j in "$JRUBY_HOME"/lib/*.jar; do
```

How come Java can't find resources in class folders that I've appended to the $CLASSPATH global variable at runtime? 
---------------------------------------------------------------------------------------------------------------------

JRubyClassLoader extends `java.net.URLClassLoader`, which treats any URL that doesn't end with a slash as a jar. See [URLClassLoader](http://java.sun.com/j2se/1.5.0/docs/api/java/net/URLClassLoader.html).

Why don't my special characters (like ção) appear correctly in my Swing app ?
-----------------------------------------------------------------------------

The file encoding for the `.rb` file containing the special characters must be set to UTF-8.

How do I check which version of the JVM JRuby is running on?
------------------------------------------------------------

```bash
    % jruby -rjava -e "puts java.lang.System.get_property('java.version')"
```

<a name="running_rails"/>
Running Rails
=============

Why does script/server (WEBrick) terminate right after saying "Booting WEBrick..."?
-----------------------------------------------------------------------------------

This happens if you require ActiveRecord-JDBC in your environment, but you haven't installed the gem. Make sure you have the ActiveRecord-JDBC gem installed.

How do I avoid installing the Rails gem twice for both CRuby and JRuby?
-----------------------------------------------------------------------

It is possible to set the GEM_HOME directory to a common one, but numerous users reporting weird behavior makes us recommend not setting a common GEM_HOME.  DO NOT DO IT :) 

The main problem arises that various gems have dependences on native gems.  If you install with MRI it will install the c-extension version and not work with JRuby.  If you install with JRuby it will not work with MRI.  Hitting one of these gems, like Nokogiri, is much too common for this to be worth it.
You just need to set GEM_HOME to point to your CRuby's gem directory.  For example, in tcsh:

I get the error "undefined method 'cattr_accessor' for ActiveRecord::Base:Class (NoMethodError)" after configuring activerecord-jdbc-adapter.  What is wrong?
-----------------------------------------------------------------------------------------------------------------------------------------------------

You're not loading AR-JDBC properly. Try requiring activerecord-jdbc-adapter in `config/environment.rb` :

```ruby
require 'activerecord-jdbc-adapter' # or require 'arjdbc'
```
If you're using Bundler this is not needed just make sure there's no `:require => nil/false` with the gem declaration :

```ruby
gem 'activerecord-jdbc-adapter' # will auto require 'activerecord-jdbc-adapter'
```

I keep getting "500.html running rails app built by goldspike in a JEE Container". What is wrong?
-------------------------------------------------------------------------------------------------

You might not have set the production DB settings in `database.yml` properly. Goldspike by default builds a war that runs the app in production mode. 

You can confirm the problem by finding the `production.log` file. If you're using JBoss, then it lives in `$JBOSS_HOME/server/my_app/tmp/my_rails_war`''1234''`.war`, where ''1234'' is a random number generated by JBoss.

In `production.log`, you'll see an error this:

```ruby
    RuntimeError (The driver encountered an error: can't convert nil into String)
```

To fix this, just set the production DB settings in `database.yml` properly.

How can I load a ResourceBundle from the Rails config directory ?
-----------------------------------------------------------------

You can add the config directory to the JRuby classpath in `config/environment.rb`:

```ruby
    require 'java'
    $CLASSPATH << "file:///#{File.expand_path(File.join(RAILS_ROOT, 'config'))}/"
```

Now, because `ResourceBundle.getBundle` doesn't seem to use the JRuby class loader, we need to pass it explicitly. In the file where you want to use the resource bundle do :

```ruby
  include Java
  require 'jruby'
  ...
  bundle = java.util.ResourceBundle::getBundle
     ("i18n", java.util.Locale::FRENCH, JRuby.runtime.jruby_class_loader)
```

Now you just need to put your `i18n[_*].properties` files in `config/`, et voilà!

<a name="calling_into_java"/>
Calling Into Java
=================

Why do I get `ClassNotFoundException` when I call `java.lang.Class.forName` from Ruby?
------------------------------------------------------------------------------------------------------------

When JRuby runs at the command line, it loads into the bootstrap class loader. This class loader lives above CLASSPATH, so libraries you add to CLASSPATH are not normally visible. Because `Class.forName` uses the class loader of the nearest non-system class that called it, it ends up seeing JRuby as the caller and looking for classes only in CLASSPATH.

The workaround for this is simple: Uuse JRuby's import or include_class methods to load the class, or just reference it directly as in `Java::my.package.MyClass` or `org.postgresql.Driver`.

How do I access an inner class with a lowercase name?
-----------------------------------------------------

Sometimes, developers name their inner classes beginning with a lowercase letter. This prevents JRuby from accessing it the "normal way" (it thinks it's a method). For example:

In Java (from the `com.sun.enterprise.ee.cms.core` package of the [Shoal project](https://shoal.dev.java.net/))

```java
    GMSConstants.shutdownType.INSTANCE_SHUTDOWN
```

Whereas in JRuby, you can get at that using the following:

```ruby
    >> ShutdownType = JavaUtilities.get_proxy_class('com.sun.enterprise.ee.cms.core.GMSConstants$shutdownType')
```

<a name="java_and_jruby_interop"/>
Java and JRuby Interoperation
=============================

 How do I create a primitive Java array in JRuby?
-------------------------------------------------

You can convert existing ruby arrays to primitive Java arrays very simply:

```ruby
    [1,2,3].to_java # makes an object array
    [1,2,3].to_java :byte # makes a byte array
    [1,2,3].to_java :String # makes a String array
```

To create empty arrays:

```ruby
    Java::byte[12].new # makes a new byte[]
    java.lang.String[12].new # makes a new String[]
```

How can I implement a Java Interface using a Ruby Class?
--------------------------------------------------------

There are two separate ways:

* Include an interface into the class.

```ruby
    class MyListener
      include java.awt.event.ActionListener

      def actionPerformed(event)
        button.text = "I have been pressed"
      end
    end
```

* Use implicit closure conversion.

```ruby
   button.add_action_listener { |event|  button.text = "I have been pressed" }
```

 Compiler-related questions
===========================

How can I compile my Ruby into Java?
------------------------------------

You can do an Ahead-Of-Time (AOT) compile by using these [[instructions|JRubyCompiler]].

Troubleshooting
===============

 Why do my ruby scripts under cygwin get executed as if they are bourne shell scripts?
--------------------------------------------------------------------------------------

Add `.bat` extension to JRuby scripts.

If you try to run JRuby on cygwin for Windows and use the commands without the `.bat` extension, it will default to the bourne shell versions, which might not work properly.

**Example using gems:***

```bash
    $JRUBY_HOME/bin/gem.bat install rails -y --no-ri --no-rdoc
```

 Why does my db:migrate fail with "undefined method 'create_database' for class '#<Class:01x1e30857>'"?
-------------------------------------------------------------------------------------------------------

In this case, try to run the migrations with:

```bash
    jruby -S rake db:migrate SKIP_AR_JDBC_RAKE_REDEFINES=1
```

You can also just put the following line somewhere in your Rakefile, or in a custom Rakefile in `lib/tasks/*.rake`:

```ruby
    ENV['SKIP_AR_JDBC_RAKE_REDEFINES'] = '1'
```

<span id="troubleshooting_JNI"></span>

 I am having a weird JNI problem. Help me!
------------------------------------------

There are two scenarios in which JNI problems occur: 

* When the jar is in JRuby 1.1.2's lib.
* When the jar is loaded using ruby's "require" statement in JRuby < 1.1.

These two scenarios are described below.

**JRuby 1.1.2 +**

JRuby 1.1.2 changed how it set up the classpath. Earlier versions merged the CLASSPATH environment variable with all .jar files in `JRUBY_HOME/lib` and passed them to the JVM via the `-classpath` option. JRuby 1.1.2 instead passes all `.jar` files in `lib` to the JVM via the `-Xbootclasspath/a:` option, while CLASSPATH is passed using `-classpath`. Jar files that use JNI (e.g. `sqljdbc.jar`) don't work correctly when they're in the boot classpath, so they should not be placed in JRuby's `lib` directory. Instead, they should be loaded with `require` or by setting the CLASSPATH environment variable.


 I tried the --server option and received an error
--------------------------------------------------

You tried the performance tuning option --server but received an error like "Error: no `server' JVM at `c:\Program Files\Java\jre6\bin\server\jvm.dll'."
On windows the Java Runtime Environment (JRE) inclues the client VM by default but does not include the server VM. You should download an install the Java Development Kit (JDK). 
If you want to use the server VM with the JRE, you can copy JDK's jre\bin\server folder to a bin\server directory in the Java SE Runtime Environment.

How can I make JRuby always use IPv4 addresses, rather than trying to use IPv6
------------------------------------------------------------------------------

Some JDKs (like OpenJDK/Sun/Oracle/Hotspot) try to use IPv6 addresses for e.g. "localhost" when possible.
If using only similar JDKs to connect, this isn't a problem. However, other tools usually prefer IPv4 addresses and may have trouble connecting.

You can adjust how the JDK (at least OpenJDK/Sun/Oracle/Hotspot) decides to use IPv6 versus IPv4 addresses
via these [networking properties](http://download.oracle.com/javase/1.4.2/docs/guide/net/properties.html).
You can add them to the Java startup options like this:

```
export JAVA_OPTS="-Djava.net.preferIPv4Stack=true"
jruby ...
```

Or pass directly to JRuby using -J-D like this:

```
jruby -J-Djava.net.preferIPv4Stack=true ...
```

I am seeing an error like "InvalidKeyException" or "Invalid key length". How can I fix it?
------------------------------------------------------------------------------------------

This error is reported when the JDK or JRE you're running on does not allow stronger encryption. There are
a few different ways to work around it.

See [[Unlimited Strength Crypto|UnlimitedStrengthCrypto]]

My server seems to have many threads stuck waiting on a Mutex, and I'm not getting good concurrency and request throughput. Why?
--------------------------------------------------------------------------------------------------------------------------------------

If you force a dump of threads on the server JVM, you may see a stack trace like this:

```
"qtp368471295-36" prio=10 tid=0x00007fbf987c6800 nid=0xa277 waiting on condition [0x00007fbf2c99c000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x000000070a1e7968> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:158)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:811)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:867)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1201)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:312)
	at org.jruby.RubyThread.lockInterruptibly(RubyThread.java:1468)
	at org.jruby.ext.thread.Mutex.lock(Mutex.java:91)
	at org.jruby.ext.thread.Mutex$INVOKER$i$0$0$lock.call(Mutex$INVOKER$i$0$0$lock.gen)
	at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:134)
	at rubyjit.Rack::Lock$$call_9D2E3A2DC6D4739AEFA0FEFD72B6C9712EF582B8.chained_0_ensure_1$RUBY$__ensure__(/data/app/taxman/installs/taxman_110bbe712d8bf6ee3955b8e87573c2cc278fe20a/vendor/bundle/jruby/1.9/gems/rack-1.4.5/lib/rack/lock.rb:14)
	at rubyjit.Rack::Lock$$call_9D2E3A2DC6D4739AEFA0FEFD72B6C9712EF582B8.__file__(/data/app/taxman/installs/taxman_110bbe712d8bf6ee3955b8e87573c2cc278fe20a/vendor/bundle/jruby/1.9/gems/rack-1.4.5/lib/rack/lock.rb)
	at rubyjit.Rack::Lock$$call_9D2E3A2DC6D4739AEFA0FEFD72B6C9712EF582B8.__file__(/data/app/taxman/installs/taxman_110bbe712d8bf6ee3955b8e87573c2cc278fe20a/vendor/bundle/jruby/1.9/gems/rack-1.4.5/lib/rack/lock.rb)
	at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:181)
	at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:168)
	at rubyjit.ActionDispatch::Static$$call_98B9D217DEE0C8798054918A3A9A20D7242072EA.__file__(/data/app/taxman/installs/taxman_110bbe712d8bf6ee3955b8e87573c2cc278fe20a/vendor/bundle/jruby/1.9/gems/actionpack-3.2.16/lib/action_dispatch/middleware/static.rb:63)
	at rubyjit.ActionDispatch::Static$$call_98B9D217DEE0C8798054918A3A9A20D7242072EA.__file__(/data/app/taxman/installs/taxman_110bbe712d8bf6ee3955b8e87573c2cc278fe20a/vendor/bundle/jruby/1.9/gems/actionpack-3.2.16/lib/action_dispatch/middleware/static.rb)
...
```

The trace above indicates there's a single JRuby runtime handling many concurrent requests, but Rails's "threadsafe" mode is not enabled. As a result, Rails inserts a lock acquisition into the request pipeline, and requests execute in serial. Generally, when deploying on JRuby, you will want to either be running threadsafe mode (single JRuby runtime, many concurrent requests) or non-threadsafe mode but with many JRuby instances.

You should modify the Rails config for your application to enable threadsafe mode: http://apidock.com/rails/Rails/Configuration/threadsafe!

Classpath errors on Mac OS X
-----------------------------

Java on OS X adds additional folders (nonstandard) to the classpath. `/Library/Java/Extensions` and `~/Library/Java/Extensions` can contain additional .jar files, which are automatically made available to Java. However, if you have a .jar in `Extensions` that also happens to be used by JRuby (but are different versions, or were compiled for an older version of Java) having both in the classpath can cause conflicts. You can examine the classpath from irb, by checking the `$CLASSPATH` global variable, which will list all .jar files used by JRuby which are in the classpath.

One common error caused by having two conflicting versions of the same class on the classpath include:

* `java.lang.VerifyError: class org.bouncycastle.asn1.ASN1Primitive overrides final method equals`

To resolve the issue, either remove the conflicting .jar from the `Extensions` directory, or swap it for a version that is compatible with the version used by JRuby.

<a name="slow_rvm_install_entropy">
Why is JRuby so slow to install via RVM?
----------------------------------------

On some Linux systems, notably including [Travis CI](http://travis-ci.org)'s Ubuntu "Trusty" image, the gemset import portion of RVM's build can proceed very slowly. This may be due to `/dev/random` entropy being too low; once exhausted, reads from `/dev/random` may block while waiting for more entropy. In a virtualized environment, this can be much slower than on direct hardware, depending on how random number generation is virtualized.

One solution is to install and enable a source of entropy like [haveged](http://www.issihosts.com/haveged/):

```
$ sudo apt-get install haveged
...
$ sudo service haveged start
```

<a name="networking"/>
Networking
==========

Why can't I connect to some (IPv4, IPv6, remote, localhost) address?
---------------------------------------------------------------

Sometimes you will have trouble connecting to a remote or local address you know is accessible and which is connectable by other means. Usually this is due to the Java runtime choosing the wrong IP version.

Most current JVMs/JDKs/JREs will try to prefer IPv6 addresses before they try IPv4 addresses. This sometimes causes trouble for JRuby applications, since they may fail trying to connect to an IPv6 address that is visible but does not expose the service you're hoping to reach. Sometimes it will be the reverse situation.

You can force OpenJDK-based runtimes (including Oracle's JDK/JRE) to prefer IPv4 by passing `-Djava.net.preferIPv4Stack` to the JVM, `-J-Djava.net.preferIPv4Stack` to JRuby, or otherwise setting the `java.net.preferIPv4Stack` property in the target JVM. The equivalent property for IPv6 `java.net.preferIPv6Stack`, but most JVMs should be doing this by default.