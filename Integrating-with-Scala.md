In JRuby 1.6.0, we shipped additional features to make it easier to integrate with Scala objects and methods. In JRuby 1.6.6, we expanded that support. This page documents what features we have added and how you can use them.

JRuby 1.6.0 - 1.6.5: Singleton support
======================================

In JRuby 1.6.0 we added logic to JRuby's Java integration layer to know when a given class was a Scala singleton. The result is that when accessing a class that has an attached singleton object, you can simply call the singleton's methods directly.

If you have a nice example of this, please add it here :)

JRuby 1.6.6: Operator aliases
=============================

As of JRuby 1.6.6, Scala operator names like $plus will gain aliased names like "+" equivalent to the symbolic form. This means that some Scala operators can be called like normal Ruby operators, while others can be sent the short symbolic name rather than the long $ name.

Names that are translated
-------------------------

Note that any combination of these names will be translated to the combined form, e.q. $plus$eq is aliased to "+=". Note also that the original $ names are still present...we just add aliases for the symbolic names.

* "$plus" becomes "+"
* "$minus" becomes "-"
* "$colon" becomes ":"
* "$div" becomes "/"
* "$eq" becomes "="
* "$less" becomes "<"
* "$greater" becomes ">"
* "$bslash" becomes "\"
* "$hash" becomes "#"
* "$times" becomes "*"
* "$bang" becomes "!"
* "$at" becomes "@"
* "$percent" becomes "%"
* "$up" becomes "^"
* "$amp" becomes "&"
* "$tilde" becomes "~"
* "$qmark" becomes "?"
* "$bar" becomes "|"

Names that can be used as normal Ruby methods
---------------------------------------------

Some names also correspond to Ruby method names and can be called directly, like a + b instead of a.send(:"$plus", b).

* a.send(:"$plus", b) is callable as "a + b"
* a.send(:"$minus", b) is callable as "a - b"
* a.send(:"$div", b) is callable as "a / b"
* a.send(:"$eq$eq", b) is callable as "a == b"
* a.send(:"$bang$eq", b) is callable as "a != b" (1.9 mode only; in 1.8 mode != compiles as negated ==)
* a.send(:"$less", b) is callable as "a < b"
* a.send(:"$less$eq", b) is callable as "a <= b"
* a.send(:"$greater", b) is callable as "a > b"
* a.send(:"$greater$eq", b) is callable as "a >= b"
* a.send(:"$times", b) is callable as "a * b"
* a.send(:"$percent", b) is callable as "a % b"
* a.send(:"$up", b) is callable as "a ^ b"
* a.send(:"$amp", b) is callable as "a & b"
* a.send(:"$bar", b) is callable as "a | b"

JRuby also maps the Ruby "[]" and "[]=" methods to Scala's "apply" and "update" methods used for () and ()= in Scala.

The following Ruby method names may also be used to call equivalent names in Scala: =~, !~ (1.9 mode only), and ===, <=>.

Unary operators are currently not mapped to any Ruby method names.

Names to watch out for
----------------------

Some operators in Scala appear to have equivalents in Ruby but do not. Don't expect the Scala behavior from these operators in Ruby (regardless of whether you're calling Scala or not)

* Any operator assignment like += is translated to the long form, e.g. a += b is executed like a = a + b.
* && and || are keywords in Ruby that only deal with the truthyness of their operands.
* Boolean assignment (||=, &&=) makes no method calls at all; it uses the || or && keyword directly, which only checks an object's truthyness. For example, a ||= b is roughly equivalent to "if (a is not nil or false); a = b; else; a; end"
* As of 1.9, the negative comparators (!=,  !~) are full methods. In 1.8 mode, they expand to the negated positive comparators.