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

# using jar-dependency as runtime-dependency and bundler

## setup

* do not pack your jars, i.e. 
* `s.add_runtime_dependency 'jar-dependencies'` to your gemspec
* `s.platform = 'java'`set platform to java 
* add jar dependencies:
```
s.requirements << 'jar org.slf4j, slf4j-api, 1.7.7'
s.requirements << 'jar org.slf4j, slf4j-simple, 1.7.7, :scope => :test'
```

now you create the `my_jars.rb` file by (assuming the name of your gem to be `my`)
```
bundle
```

this will install the jar into the local maven repository and create the my_jars.rb file.

everything else now works as above.

## pros and cons

in the past there was problems with debian openjdk8 package which did not install the ca-certs in the java keystore and could not talk to maven central. the blame went naturally to the jar-dependencies project.

all in all vendoring jars within the gem is the recommended way of doing things, as the eco-system of gems with such jars using jar-dependencies is not so huge to playout the pros and it is unlikely to run into extra trouble while installing the gem.

### pros

- no jars get vendored inside the gem
- gems and jars are more or less treated alike
- the jars get shared via the local maven repo between projects and/or installed gems

### cons

- any problem with installing the jars during gem install is very hard to debug
- any restricted access to the rubygems respository server needs to be accompanied by maven-repository server with the same access rights.
- proxy support for downloading jar might be buggy

