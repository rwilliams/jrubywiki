This page provides the name mangling specification for the JRuby compiler.

**Given the following packaging specification:**

* Each element of the package must be cleansed to contain only valid Java package characters.
* Each element of the resulting class name must be cleansed to contain only valid Java classname characters.
* The resulting filename should match the class name, to allow Java classloading to be used to load it in more traditional scenarios.
* The class name (matching file name) must be reversible, consistent, and predictable for all automatic cases, to allow the Ruby load process to accurately locate the class files in every case.
* The package name does not have to match the eventual location in the load path, and since it must be examined for every compiled `.class `load, it must be valid only  for Java's purposes (and not necessarily reversible or predictable for ours).

'''Methods on `java.lang.Character` can be used to clean up the package and class name on a character-by-character basis.'''

Method names must conform to Java method naming. This primarily impacts method names containing characters that Java does not support, such as arithmetic operators, question marks, bangs, and equality/comparison symbols. Below is a complete list of such names and what they should translate to:

:`+ `will be` "op_plus"`<br/>
:`- `will be` "op_minus"`<br/>
:`* `will be` "op_times"`<br/>
:`/ `will be` "op_divide"`<br/>
:`** `will be` "op_pow"`<br/>
:`== `will be` "op_equal"`<br/>
:`=== `will be` "op_eqq"`<br/>
:`eq? `will be` "eq_p"`<br/>
:`< `will be` "op_lt"`<br/>
:`<= `will be` "op_le"`<br/>
:`> `will be` "op_gt"`<br/>
:`>= `will be` "op_ge"`<br/>
:`<=> `will be` "op_cmp"`<br/>
:`something! `will be` "something_bang"`<br/>
:`[] `will be` "op_aref"`<br/>
:`something= `will be` "something_set"`<br/>
:`[]= `will be` "op_aset"`<br/>
