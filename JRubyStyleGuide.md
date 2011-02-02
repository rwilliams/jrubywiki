When contributing code or patches to JRuby, please try to follow this style guide. Of course everyone has their own style they like to use on their personal code, but using a conflicting style in contributions just makes more work for the committers. Keeping the code clean and consistent makes it easier to maintain.

Java Code
---------

* if statements with no braces are fine, if they stay on the same line. NO braceless if statements that span lines. If you have an else block, use braces everywhere...it's too confusing without.
* single space after commas in parameter lists. In general, follow standard Sun Java style guide for whitespacing.
* 4 space indenting. 4-8 additional spaces for wrapped lines, depending on how it looks...nothing strict here. Absolutely no tabs.
* use whitespace appropriately to offset sections of code. If you have more than ten lines of code with no whitespace breaking it up, you need to add some in logical places. It's much harder to follow a long sequence of lines stacked one on top of the other than a few chunks that each logically fit together
* ternary operators are ok if they're reasonable. Absolutely no nested ternary operators (ugh!).
* use long, descriptive identifiers. no single-letter or letter-number names. We have to rename them to something readable in many cases.

Ruby Code
---------

* two space indent
* avoid wrapping statements if possible, due to the ambiguity of wrapped statements in Ruby. If you must wrap, be sure to offset by something like four spaces and TRIPLE CHECK the logic is right.
* single space after commas on param lists
* method definitions use parens, always. Of course you can do whatever you want in your own code, but for JRuby code we need it consistent.
* always use && and ||, not 'and' and 'or'
* follow Ruby naming standards...avoid camelCase except where necessary for Java, underscores between words, SHORT names when possible, short descriptive identifiers.
* (debatable) prefer do...end for any blocks that can't fit on a single line? again, consistency.
* DO NOT write code that depends on ObjectSpace, Thread#critical or kill or raise, trace functions (except where needed for debugging tools, since none are available), or other POSIX functions we have only hacky impls for.

