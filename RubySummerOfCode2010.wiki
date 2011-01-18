This page is for ideas for the Ruby Summer of Code, in which JRuby will participate. Please feel free to add your idea (plus a description and whether you'd be willing to work on it (as a student) or mentor for it!

Here's some broad categories to get you started:

* Native libraries that need a Java port (or wrap a Java lib?)
** Pure Ruby/Java Nokogiri support: provide a package of Nokogiri for the Java platform which does not need FFI or access to native libxml2/libxslt libraries but instead uses the native Java XML libraries.  In addition to the RSoC compensation for students, a [http://wiki.github.com/rdp/ruby_bounties/ruby-bounties $650 bounty is also available]
* Work on the JRuby C Extension support
** Wayne Meissner has started a C extension module for JRuby here: http://github.com/wmeissner/jruby-cext. Someone with knowledge of the Ruby C APIs (or interested in them) could help us build out a "safe" set of compatible APIs that call through JNI into JRuby.
* JRuby on Android: Ruboto
** There's already an IRB for Android, but we'd like to do a more concentrated effort to make JRuby work well on Android devices. There's a need for tooling (project generation, nice commands like Rails), performance work (find the right combination of flags, compilation, and tweaks to JRuby), and code generation (being able to generate some .class files up front for Android to use). We want JRuby to be a full Android language!
* A new fast tiny server based on the GlassFish gem, or else taking over the GF gem and spinning it off into its own project.
** The GlassFish gem is great, but there's now nobody at Oracle responsible for working on it. We need to either take it over (by spinning it off and simplifying it) or we need to come up with one that we can maintain ourselves. There's definitely a need for a fast, tiny, gem-installable server.
* Hibernate/JPA support for DataMapper or DataObjects
** If DataMapper supported Hibernate or JPA, we'd finally have a dead simple, clean, and powerful persistence API for Ruby that works with JRuby and works with dozens of databases. Hibernate is one of the most impressive ORMs available for Java, and putting a DataMapper API on the front would be amazing.
* Duby IDE support
** The Duby programming language is like a Ruby face for Java (and other languages). It's statically type and has a very simple toolchain. We'd love to see work done to support Duby in an IDE, specifically to use Duby's type inference to provide method completion and static-typed error checking like Java has. You would be able to start with a Ruby parser and go from there. NetBeans or Eclipse are welcome.
* JRuby internals work: hook up optimizing compiler, improve some class or library
** Subbu Sastry has been working on an optimizing compiler for JRuby for some time, but we have never had resources to start trying to hook it up to bytecode generation. Someone familiar with compilers and JVM bytecode could take on a project to get some amount of Ruby code running with the new compiler.
** Change the way file descriptors are handled. Currently most file descriptors in JRuby are fake ones, but these fake file descriptors can cause conflicts when one wants to instantiate an I/O object from a native OS file descriptor, e.g. with UNIXSocket.for_fd. There should be some way to distinguish fake from real file descriptors just by looking at their values, e.g. by numbering fake file descriptors starting from the file descriptor limit.
** Besides generating bytecode, another standalone project is ability to write / read bytecode to / from disk (".s" files in C parlance).  For example, ruby code from standard libraries (that are either implemented in ruby or in java) would have these IR equivalents which will be used during optimization phases (ex: inlining).   This eliminates the need to recompile standard libraries over and over again and would be the jruby equivalent of pre-compiled libraries (.jar, .so).  This project doesn't require strong compiler or jvm bytecode knowledge / skills.  Initial work would be to generate text (human-readable / editable) versions.  If necessary, in a later pass, can generate binary equivalents (which would be an optimization to reduce file IO and IR parsing times).
* Adopt-a-server: pick a Java or JRuby server (JBoss, WebLogic, WebSphere, GlassFish, GlassFish Gem, Tomcat, general Java EE) and build out support for it
* an auto ruby -> duby [http://www.ruby-forum.com/topic/206733#900136 compiler]
* WIN32OLE for JRuby
** JRuby runs consistently and fast across multiple platforms, but it needs some love on Windows. Build on the work already started by others to add the JACOB library for WIN32OLE support (see the [http://github.com/bpmcd/win32ole bpmcd-win32ole github project]).
* Build out tooling for Maven
** JRuby 1.5 adds great integration of Ant and Rake, but there's still a need for better support for Maven. Some projects have already been started like the various maven plugibs for Rails, RubyGems and more. There's also the beginnings of maven build support for maven 3's "polyglot maven" project. But there's a lot more work to doo to make JRuby fit perfectly into Maven projects.
* Fix the slowdown that Jruby experiences with [http://wiki.github.com/rdp/ruby_bounties/ruby-bounties#jruby_slowdown rails].
* There are some more things listed on the [http://wiki.github.com/rdp/ruby_bounties/ruby-bounties ruby bounties] page.
