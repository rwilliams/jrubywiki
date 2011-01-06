JRuby 1.2 was the first release of JRuby to officially include large portions of Ruby 1.9's feature set. Work has continued, piecemeal, over releases since then. The current plan as of December 2010 is to have JRuby 1.6 be the first release where we really expect people to start using 1.9 support and reporting bugs against it. That release is slated for at least an RC in December 2010, and likely a final release sometime after January 1, 2011.

There are two areas people really could pitch in: writing specs and helping with implementation.

=Writing Specs=

RubySpec has improved substantially over the past 6-9 months, due to a push by the 1.9 maintainer Yuki Sonoda to start using RubySpec for MRI development. As a result, we now have at least a good starting point to use when verifying 1.9 compatibility.

There are still many weak areas, however. If you're interested in contributing RubySpecs, check out http://www.rubyspec.org for information on how to help out.

In general, the first step to finding a JRuby 1.9 feature to help implement would be looking at spec/tags/1.9 subdirs in JRuby, reading the associated spec, and implementing the missing or broken behavior.

=Code conventions=

Sometimes we just need to change some code of a method to get it 1.9 compliant. There are a few conventions we have to follow to adapt those methods.

* We should avoid to check runtime mode by wrapping 1.9 methods around 1.8 ones:

  '''bad'''
  @JRubyMethod
  public IRubyObject collect(ThreadContext context, Block block) {
    if (block.isGiven()) {
      //do some cool
    }  else if (context.getRuntime().is1_9()) {
      // return Enumerator
    } else {
      // return Array
    }
  }

  '''good'''
  @JRubyMethod(name = "collect", compat = CompatVersion.RUBY1_8)
  public IRubyObject collect(ThreadContext context, Block block) {
    return block.isGiven() ? collectCommon(context, block) : newArray();
  }
  @JRubyMethod(name = "collect", compat = CompatVersion.RUBY1_9)
  public IRubyObject collect19(ThreadContext context, Block block) {
    return block.isGiven() ? collectCommon(context, block) : newEnumerator();
  }

* When we have two methods for different modes, ie Ruby 1.8 and 1.9, we have to add the version number without punctuations, dashes or underscores as the suffix of the method name for 1.9 mode. I.e: '''collect''' and '''collect19'''.

=Implementation=

Here's the status of Ruby 1.9 feature support and work remaining:

==Done==

These are done but need more testing. Where noted, pieces are missing.

* Array
** missing encoding aware pack ([http://jira.codehaus.org/browse/JRUBY-3426 JRUBY-3426])
** sort should obey modified Fixnum and String class (but 1.9 still optimizes these case) ([http://jira.codehaus.org/browse/JRUBY-3427 JRUBY-3427])
* Hash
* String
** missing encoding aware % ([http://jira.codehaus.org/browse/JRUBY-3428 JRUBY-3428]), unpack ([http://jira.codehaus.org/browse/JRUBY-3429 JRUBY-3429]), crypt ([http://jira.codehaus.org/browse/JRUBY-3430 JRUBY-3430])
** missing encode/decode (via Encoding::Converter) ([http://jira.codehaus.org/browse/JRUBY-3431 JRUBY-3431])
** possible gotchas with to_sym
** string constructions in multiple places in core should taint/set coderanges and encoding
* Symbol
** intern gotchas due to encoding information in symbols (needs to be done)
** might loose encodings in some cases
* Regexp
* Encoding
** missing default_internal/default_external ([http://jira.codehaus.org/browse/JRUBY-3433 JRUBY-3433])
** missing system encoding guess ([http://jira.codehaus.org/browse/JRUBY-3434 JRUBY-3434])
* Complex
* Rational
* Numeric
* Float
** remaining rubyspec deals with formatting to_s properly, which we've always had some issues with (even in 1.8 mode)
* Math
* Mathn
* Range
* Struct
* Kernel (mostly)
** Some of the remaining RubySpec failures are shared with 1.8 mode.
* Method
* Module
* Thread?
* Fiber?

==Missing==

These are entire areas that need to be worked on.

* Encoding::Converter
* Yielder/Generator
** At least partially implemented for 1.8.7 support, but it's unknown whether the impl is sufficient for 1.9 mode.
* key Marshal changes
* cli options
* some RubyBignum changes
* possible other changes in Numerics
* changes in Dir/IO/File (some obvious things are done, like enumeratorize)
* encoding information in exception messages (now passed via java String)
* BigDecimal changes ?
* Symbol to be represented as encoded bytes, rather than a UTF-16 Java String (wide-reaching implications, since we use String many places where a "string key" is needed, like in method and variable tables).