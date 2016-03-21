JRuby uses the [spec](https://github.com/ruby/spec) project for compatibility testing, along with our own (somewhat dated) suites of tests.

To run the tests 

`ant spec:ci_interpreted_19` 

or 

`ant spec:ci_interpreted_18`

There are other [`ant` tasks][ant-tasks] that can be used to run variations of the RubySpecs, which point to their equivalent [Rake tasks][rake-spec].

We generally prefer that people adding new tests for Ruby behavior submit them to the RubySpec project, so that all Ruby implementations benefit from those additions. For more information on how to contribute to RubySpec, please visit the page below:

[ruby/spec](https://github.com/ruby/spec)

[mspec]: http://rubyspec.org
[ant-tasks]: https://github.com/jruby/jruby/blob/586f44f2543c4cdd0e5cbb345517aaf87af1098b/build.xml#L1177-L1216
[rake-spec]: https://github.com/jruby/jruby/blob/586f44f2543c4cdd0e5cbb345517aaf87af1098b/rakelib/spec.rake#L22-L66