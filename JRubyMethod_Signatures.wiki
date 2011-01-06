[[Home|&raquo; JRuby Project Wiki Home Page]] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [[Internals|&raquo; Design: Internals]] 
<h1>JRuby Method Binding</h1>
<tt>JRubyMethod</tt> allows specifying method signatures, like required, optional, and rest arguments, using normal Java signatures. In most cases, the method binding logic in JRuby will handle all argument counts, arity checking, and argument passing for you. This document describes the allowed combination of signatures.

==Rules==
* <tt>ThreadContext</tt> is optional and must always come as the first argument.
* <tt>Block</tt> is optional and must always come as the last argument.
* If the Java method is static, it must have a <tt>self</tt> argument as the first argument or immediately following <tt>ThreadContext</tt>.
* Specific argument counts are supported for 0 to 3 arguments. All signatures that require more than 3 arguments must use an argument array.
* If there are multiple methods that are named the same but take different argument counts, they will be used to provide separate methods for optional arguments.
* If a method has required arguments but also accepts a rest arg, you should either specify it as an argument array or specify all argument counts up to the required count plus an argument version.

So a rough specification for the signature would be as follows:

  public [static] IRubyObject methodname(
      [ThreadContext context, ]
      [IRubyObject self, ]
      <nowiki>[[IRubyObject arg0 [, ... IRubyObject arg2]] | IRubyObject[] args]
      [, Block block]) { ...</nowiki>

==Examples==

* One required argument, no <tt>ThreadContext</tt>, no <tt>Block</tt>

    @JRubyMethod
    public IRubyObject method(IRubyObject arg0) {

* Two required arguments, static, <tt>ThreadContext</tt>, no <tt>Block</tt>

    @JRubyMethod
    public static IRubyObject method(ThreadContext context, IRubyObject self, IRubyObject arg0) {

* One required argument, no <tt>ThreadContext</tt>, no <tt>Block</tt>, with rest arg

    // this is called if only no args are passed
    @JRubyMethod
    public static IRubyObject method() { ... // you should raise error here

    // this is called if one arg is passed
    @JRubyMethod
    public static IRubyObject method(IRubyObject arg0) { ...

    // this is called if two or more args are passed
    @JRubyMethod
    public static IRubyObject method(IRubyObject[] args) { ...

* One required argument, one optional argument, no <tt>ThreadContext</tt>, no <tt>Block</tt>

    // this is called if one argument is passed
    @JRubyMethod
    public static IRubyObject method(IRubyObject arg0) { ...

    // this is called if two arguments are passed
    @JRubyMethod
    public static IRubyObject method(IRubyObject arg0, IRubyObject arg1) { ...
