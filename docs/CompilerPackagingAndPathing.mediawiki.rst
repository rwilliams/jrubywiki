[[Home|&raquo; JRuby Project Wiki Home Page]]  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [[Internals|&raquo; Design: Internals]]
=Compiler Packaging and Pathing=
This is the packaging and pathing specification for JRuby compiler.

'''Given:'''

* A <tt>baz.rb</tt> file exists in some directory <tt>foo/bar</tt>, which exists under user's home directory.

'''Requirements:'''

* Compilation of <tt>baz.rb</tt should produce <tt>baz.class</tt> in the same location.
* Java's requirement for unique class names means <tt>baz.class</tt> must have a unique class name.
* <tt>baz.class</tt> may be executed from locations other than the one from which it was compiled.

So assume <tt>baz.class</tt> contains a class named <tt>foo.bar.baz</tt>, existing under <tt><nowiki><home>/foo/bar</nowiki></tt>.

* Load path contains <tt><home></tt>.
* Executing <tt>require 'foo/bar/baz'</tt> should pick up the class file and load it.
* Executing <tt>jruby foo/bar/baz.class</tt> should execute it correctly.
* Executing <tt>jruby -e "require 'baz'"</tt> from within <tt>&lt;home&gt;/foo/bar</tt> should pick it up correctly.

Load path locations are typically expected by the source; <tt>require</tt>s and such depend on their being correct. If the load path points at <tt>&lt;home&gt;</tt> normally, pointing it at <tt>&lt;home&gt;/foo</tt> instead would likely cause <tt>require</tt>s in <tt>baz</tt> to fail. So:
* We can reasonably expect that the load path structure is a good representation of a unique class name, and that execution would not expect something more unique. 
* Therefore, a subpath within the load path is a good enough indicator of package for the compiled class.
* However, load path searching should still be done. If we switch to <tt>foo/bar</tt> and try to require <tt>baz</tt>, it should continue to load. If we try to run <tt>baz</tt> directly, it should continue to load.
* Therefore, we should continue to use load path searching as the primary mechanism for finding the compiled class file.
* Therefore, we should also examine the class file directly for the contained class name, rather than expecting it to match package structure.

'''Given that compilation may happen from any arbitrary directory:'''

We need a way to determine the appropriate load path for a file under compilation. Because load paths may differ at runtime for the script/app in question, we should do three things:

* Provide a command-line means to specify the base dir from which the source is being compiled. This is similar to the requirement in <tt>.java</tt> files to specify a host dir that represents the "root" of the path.
* Default to current directory as the "root" of the path, if the filename to be compiled does not contain relative path modifiers and is a subpath under the current directory.
* If no base is given and there are relative modifiers in the full file path, generate a package name based on the full canonical filename. Perhaps print a warning.

'''Addendums:'''

* For any given file that must use a canonical path containing a device name to generate package name, the device indicator (<tt>C:\</tt>, <tt>\\somehost\</tt>, etc.) will be considered the root and omitted from the resulting package.

'''A few cases:'''

'''1. Simple Case'''
:current dir<nowiki>:</nowiki> <tt>C:\home</tt>
:file to compile<nowiki>:</nowiki> <tt>C:\home\foo\bar\baz.rb</tt>, specified as <tt>foo\bar\baz.rb</tt>
:no basedir specified

* filename given has no relative modifiers
* filename is within a subpath of current directory
* package generated is <tt>foo.bar</tt>
* <tt>baz.class</tt> file is placed in <tt>C:\home\foo\bar\baz.class</tt>

'''2. Second Case'''
:current dir<nowiki>:</nowiki> <tt>C:\home2</tt>
:file to compile<nowiki>:</nowiki> <tt>C:\home\foo\bar\baz.rb</tt>, specified as <tt>..\home\foo\bar\baz.rb</tt>
:no basedir specified

* filename has relative elements
* canonical path is <tt>C:\home\foo\bar\baz.rb</tt>
* package generated is <tt>home.foo.bar</tt>

'''3. Third Case'''
:current dir<nowiki>:</nowiki> <tt>C:\home2</tt>
:file to compile<nowiki>:</nowiki> <tt>C:\home\foo\bar\baz.rb</tt>, specified as<tt>C:\home\foo\bar\baz.rb</tt>
:basedir<nowiki>:</nowiki> <tt>C:\home</tt>

* basedir is specified
* target file is in a subpath of basedir with no relative modifiers
* package and result same as in (1) Simple Case above

'''Execution cases that should succeed in loading a<tt> baz.class</tt>, given any of the results above''

'''Case 1''
:load path contains <tt><nowiki>C:\home</nowiki></tt>
:<tt>jruby -e "require 'baz'"</tt>

'''Case 2'''
:load path contains <tt>C:\home2</tt>
:current dir is <tt>C:\home2</tt>
:<tt>jruby -e "require '../home/foo/bar/baz'"</tt>

'''Case 3'''
:current dir is <tt>C:\home2</tt>
:<tt>jruby ../home/foo/bar/baz.class</tt>
