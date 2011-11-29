JRuby uses the RubySpec project for compatibility testing, along with our own (somewhat dated) suites of tests.

To run the tests 

`ant spec:ci_interpreted_19` 

or 

`ant spec:ci_interpreted_18`

We generally prefer that people adding new tests for Ruby behavior submit them to the RubySpec project, so that all Ruby implementations benefit from those additions. For more information on how to contribute to RubySpec, please visit the page below:

[The RubySpec Project](http://rubyspec.org)