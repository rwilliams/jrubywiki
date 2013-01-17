Clone and build JRuby on any Java 6+ JDK.

```
$ git clone https://github.com/jruby/jruby.git jrubyx
Cloning into 'jrubyx'...
remote: Counting objects: 210154, done.
remote: Compressing objects: 100% (51291/51291), done.
remote: Total 210154 (delta 145665), reused 208466 (delta 144170)
Receiving objects: 100% (210154/210154), 113.77 MiB | 609 KiB/s, done.
Resolving deltas: 100% (145665/145665), done.

$ cd jrubyx

$ ant clean jar
```

Install gems needed for development (rake, rspec, etc).

```
$ ant install-dev-gems
Buildfile: /Users/headius/projects/jrubyx/build.xml
...
install-dev-gems:
     [java] Gem.ruby /Users/headius/projects/jrubyx/bin/jruby
     [java] Gem.bindir /Users/headius/projects/jrubyx/bin
     [java] INFO:  gem "rake" is not installed
     [java] INFO:  gem "rspec-core" is not installed
     [java] INFO:  gem "diff-lcs" is not installed
     [java] INFO:  gem "rspec-expectations" is not installed
     [java] INFO:  gem "rspec-mocks" is not installed
     [java] INFO:  gem "rspec" is not installed
     [java] INFO:  gem "minitest" is not installed
     [java] INFO:  gem "minitest-excludes" is not installed
     [java] Successfully installed rake-10.0.2
     [java] Successfully installed rspec-core-2.8.0
     [java] Successfully installed diff-lcs-1.1.3
     [java] Successfully installed rspec-expectations-2.8.0
     [java] Successfully installed rspec-mocks-2.8.0
     [java] Successfully installed rspec-2.8.0
     [java] Successfully installed json-1.7.3-java
     [java] Depending on your version of ruby, you may need to install ruby rdoc/ri data:
     [java] 
     [java] <= 1.8.6 : unsupported
     [java]  = 1.8.7 : gem install rdoc-data; rdoc-data --install
     [java]  = 1.9.1 : gem install rdoc-data; rdoc-data --install
     [java] >= 1.9.2 : nothing to do! Yay!
     [java] Successfully installed rdoc-3.12
     [java] Successfully installed minitest-2.11.0
     [java] Successfully installed minitest-excludes-1.0.0
     [java] 10 gems installed

BUILD SUCCESSFUL
Total time: 26 seconds
```

Running the base test suite happens via `ant test`. Longer suites are available via rake targets.

* Base suite (interpreted): `ant test`
* JIT-forced and AOT-forced JRuby suites: `rake test:jruby19:jit` or `rake test:jruby19:aot`
* JIT-forced and AOT-forced MRI 1.9 suites: `rake test:mri19:jit` or `rake test:mri19:aot`

If running on Hotspot v34+ invokedynamic will be *on* by default. Running on earlier versions of Hotspot require passing a flag to JRuby, which can be done for test runs using an env var:

```
$ JRUBY_OPTS=-Xcompile.invokedynamic=true rake test:mri19:aot
JAVA options: {:fork=>"true", :failonerror=>"true", :classname=>"org.jruby.Main", :maxmemory=>"1024M"}
...
```

Benchmarks are available in the [rubybench project](https://github.com/jruby/rubybench), and can be used to test performance and JIT.