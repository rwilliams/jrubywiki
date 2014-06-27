## setup for minispec

first make sure you have Ruby-Maven installed

    gem install ruby-maven

you need a gemspec-file or **Gemfile** for your project. now you need a **Mavenfile** like this

    gemfile

    jruby_plugin :minitest do
      execute_goals( :spec )
    end
	
    properties( 'jruby.versions' => ['1.5.6','1.6.8','1.7.13'].join(','),
                'jruby.modes' => ['1.8', '1.9', '2.0'].join(',') )

with this file in place you can run your tests with

    rmvn test

this matrix is **not** the product of those to arrays/vectors the minitest-plugin takes care that it runs only the allowed tests. so the specs will run with

    jruby-1.5.6 mode 1.8
    jruby-1.6.8 mode 1.8
    jruby-1.6.8 mode 1.9
    jruby-1.7.13 mode 1.8
    jruby-1.7.13 mode 1.9
    jruby-1.7.13 mode 2.0

to run minitest instead of minispec as in the above example change to

    jruby_plugin :minitest do
      execute_goals( :test )
    end

## rspec

the same thing works for rspec like this

    gemspec

    jruby_plugin :rspec do
      execute_goals( :test )
    end

    snapshot_repository :jruby, 'http://ci.jruby.org/snapshots/maven'

    properties( 'jruby.versions' => ['1.5.6','1.6.8','1.7.13','9000.dev-SNAPSHOT'].join(','),
                'jruby.modes' => ['1.8', '1.9', '2.0','2.1'].join(','),
                'tesla.dump.pom' => 'pom.xml',
                'tesla.dump.readonly' => true )

here the Mavenfile looks for gemspec0file and resolves and installs the dependencies for it and then execute the rspec in context of these gems. further it adds JRuby-9000.dev to list of JRubies - note the extra **2.1** in the modes !

note that there are two more "magic" properties which tells Ruby-Maven to dump a **pom.xml** which can be used by proper Maven with exact the executions as with Ruby-Maven.

## cucumber

there is also a Cucumber JRuby-Plugin:

    jruby_plugin :cucumber do
      execute_goals( :test )
    end

## using it on travis

the pom.xml can be used to setup travis as such (.travis.yml)

    language: ruby
    rvm:
      - 1.9.3
      - 2.0.0
      - 2.1.0
      - ruby-head
    matrix:
      include:
        - rvm: jruby
          jdk: openjdk6
          script: mvn test
        - rvm: jruby
          jdk: openjdk7
          script: mvn test
        - rvm: jruby
          jdk: oraclejdk7
          script: mvn test
        - rvm: jruby
          jdk: oraclejdk8
          script: mvn test
      allow_failures:
        - rvm: ruby-head

which runs the usual ```bundle exec rake``` for native rubies and for jruby it runs ```mvn test``` which uses the above mentioned **pom.xml** and runs over configured matrix of the **Mavenfile**. as you see the test run with different JDKs.