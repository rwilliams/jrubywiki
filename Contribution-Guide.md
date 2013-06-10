
# Testing

Here are the different test commands:

* `ant test` : runs all JRuby unit tests.
* `ant spec-short` : runs Rubyspecs for 1.8 and 1.9 in interpreted mode.

## Rubyspecs

* To run a spec separately:

```
./bin/jruby -S mspec /path/to/spec.rb
```
* Do not change specs in the JRuby codebase, as they are being updated from https://github.com/rubyspec/rubyspec/.  Instead create a PR in the Rubyspecs project.

# Pull Requests

* Use feature branch if possible (?)
* Keep big formatting changes in separate commits or PR to make review easier.
* See the [GitHub Guide](https://help.github.com/articles/using-pull-requests) for more details.

# Coding Conventions 

See the [JRuby Style Guide](https://github.com/jruby/jruby/wiki/JRubyStyleGuide).