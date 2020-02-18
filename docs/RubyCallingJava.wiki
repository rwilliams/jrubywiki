This page describes the logic used for dispatching from Ruby to Java, including how methods are selected and how users can modify the process from Ruby.

==Background==

JRuby allows calling Java classes from Ruby by wrapping those classes with Ruby-friendly class/module structures we call "proxies". The proxies look and feel like normal Ruby classes or modules, and can be modified, monkey-patched, and re-opened. All methods of a given name in the Java class are bound to a single Ruby method of the same name.

For Java methods with no overloads, the dispatching process is simply to coerce passed arguments to the types of the one target method. For Java methods with overloads, the process is more complicated, and a "best match" is selected before proceeding with coercion.

==JRuby 1.1.5-1.3.1 Dispatch Logic==

For releases from JRuby 1.1.5 through JRuby 1.3.1, the method selection logic worked like this:

For each callable...
* try to find a 100% exact match using class.getJavaClass (only one allowed per type)
* try to find an assignable&primitivable match using JavaClass.assignable and type-specific checks for numerics
* try to find an assignable&duckable match using JavaClass.assignable and JavaUtil.isDuckTypeConvertible

If any of the above finds results, we calculate an "exactness" score and the remaining searches do not fire. The most exact match in a given step is chosen as the target.

Exactness is calculated based on how many of the target callable's parameter types exactly matches that returned from class.getJavaClass. In the case of a "100% exact match" search, obviously the successful result is the most exact automatically.

==Selecting a Target Method==

==Coercing Values==

==Overriding Coercion or Selection Behavior==

==Forcibly Casting Values==

==Troubleshooting==