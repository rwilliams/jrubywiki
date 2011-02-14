[[Home|&raquo; JRuby Project Wiki Home Page]]  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [[Internals|&raquo; Design: Internals]]
=Compiler Name Managing=
This page provides the name mangling specification for the JRuby compiler.

'''Given the following packaging specification:'''

* Each element of the package must be cleansed to contain only valid Java package characters.
* Each element of the resulting class name must be cleansed to contain only valid Java classname characters.
* The resulting filename should match the class name, to allow Java classloading to be used to load it in more traditional scenarios.
* The class name (matching file name) must be reversible, consistent, and predictable for all automatic cases, to allow the Ruby load process to accurately locate the class files in every case.
* The package name does not have to match the eventual location in the load path, and since it must be examined for every compiled <tt>.class </tt>load, it must be valid only  for Java's purposes (and not necessarily reversible or predictable for ours).

'''Methods on <tt>java.lang.Character</tt> can be used to clean up the package and class name on a character-by-character basis.'''

Method names must conform to Java method naming. This primarily impacts method names containing characters that Java does not support, such as arithmetic operators, question marks, bangs, and equality/comparison symbols. Below is a complete list of such names and what they should translate to:

:<tt>+ </tt>will be<tt> "op_plus"</tt><br/>
:<tt>- </tt>will be<tt> "op_minus"</tt><br/>
:<tt>* </tt>will be<tt> "op_times"</tt><br/>
:<tt>/ </tt>will be<tt> "op_divide"</tt><br/>
:<tt>** </tt>will be<tt> "op_pow"</tt><br/>
:<tt>== </tt>will be<tt> "op_equal"</tt><br/>
:<tt>=== </tt>will be<tt> "op_eqq"</tt><br/>
:<tt>eq? </tt>will be<tt> "eq_p"</tt><br/>
:<tt>< </tt>will be<tt> "op_lt"</tt><br/>
:<tt><= </tt>will be<tt> "op_le"</tt><br/>
:<tt>> </tt>will be<tt> "op_gt"</tt><br/>
:<tt>>= </tt>will be<tt> "op_ge"</tt><br/>
:<tt><=> </tt>will be<tt> "op_cmp"</tt><br/>
:<tt>something! </tt>will be<tt> "something_bang"</tt><br/>
:<tt>[] </tt>will be<tt> "op_aref"</tt><br/>
:<tt>something= </tt>will be<tt> "something_set"</tt><br/>
:<tt>[]= </tt>will be<tt> "op_aset"</tt><br/>
