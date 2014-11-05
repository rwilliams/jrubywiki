When contributing code or patches to JRuby, please try to follow this style guide. Of course everyone has their own style they like to use on their personal code, but using a conflicting style in contributions just makes more work for the maintainers. Keeping the code clean and consistent makes it easier to maintain.

Java Code
---------

* `if` statements with no braces are fine, if they stay on the same line. Do not use braceless `if` statements that span multiple lines. If you have an `else` block, use braces everywhere.
* Use single space after commas in parameter lists. In general, follow the [standard Sun Java style guide for whitespacing](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-141388.html#475).
* 4 space indenting. 4-8 additional spaces for wrapped lines, depending on how it looks; nothing strict here. **Absolutely no tabs**.
* Max line length is 110 characters.
* Use whitespace appropriately to offset sections of code. If you have more than ten lines of code with no whitespace breaking it up, you need to add some in logical places. It's much harder to follow a long sequence of lines stacked one on top of the other than a few chunks that each logically fit together
* Ternary operators are ok if they're reasonable. Absolutely no nested ternary operators (ugh!).
* Use long, descriptive identifiers. No single-letter or letter-number names. We have to rename them to something readable in many cases. Avoid abbreviations in favor of full variable names when possible.

Ruby Code
---------

* Two space indent
* Avoid wrapping statements if possible, due to the ambiguity of wrapped statements in Ruby. If you must wrap, be sure to offset by something like four spaces and TRIPLE CHECK the logic is right.
* Single space after commas on param lists
* Method definitions use parentheses.
* Always use && and ||, not 'and' and 'or'
* Follow Ruby naming standards...avoid camelCase except where necessary for Java, underscores between words, SHORT names when possible, short descriptive identifiers.
* (debatable) prefer do...end for any blocks that can't fit on a single line? again, consistency.
* **Do not** write code that depends on `ObjectSpace`, `Thread#critical`, `Thread#kill`, `Thread#raise`, trace functions (except where needed for debugging tools, since none are available), or other POSIX functions we have only hacky implementations for. See [this blog post](http://blog.headius.com/2008/02/rubys-threadraise-threadkill-timeoutrb.html) for more information.

