## create gemspec file ##

minimum for using jars (my.gemspec) adding a runtime jar dependency (slf4j-api) and a test jar dependency (slf4j-simple):

* set platform to java `s.platform = 'java'` - otherwise the requirements will be ignored
* add jar dependencies:
```
s.requirements << 'jar org.slf4j, slf4j-api, 1.7.7'
s.requirements << 'jar org.slf4j, slf4j-simple, 1.7.7, :scope => :test'
```
* ensure you pack the jars `s.files += Dir['lib/**/*.jar']`

## vendor jars inside the gem (recommended) ##

if your installed jar-dependencies has this executable already then just run it
```
vendor_jars
```
otherwise execute
```
jruby -rjars/installer -e 'Jars::Installer.vendor_jars!'
```

this will vendor all jars from the requirements and its transitive dependencies under `lib` (or what ever `s.require_path` was configured in the gemspec). it also creates a file `lib/my_jars.rb` where `my` is the name of you gem. require this file `require 'my_jars'` and all jars will be added to the classloader. this is actually important to play nice with other gems doing the same. on version conflict this ensures only one version gets loaded and that you see a warning to resolve the versions conflict by using `lock_jars`

the test jars are install into the local maven repository and can be used for tests.

`jar-dependency` is a default gem in jruby and thus there is no need to have an explicit dependency on it.

## example project ##

See [gem-with-jar-dependencies](https://github.com/mkristian/jar-dependencies/tree/master/examples/gem-with-jar-dependencies) for working example.