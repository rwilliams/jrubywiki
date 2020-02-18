''This page is being developed. For now it is merely a list of tasty links.''

__TOC__

== ci_reporter ==
''CI::Reporter is an add-on to Test::Unit and RSpec that allows you to generate XML reports of your test and/or spec runs. The resulting files can be read by a continuous integration system that understands Ant's JUnit report XML format, thus allowing your CI system to track test/spec successes and failures.''

* [http://blog.nicksieger.com/articles/2007/01/06/continuous-integration-goodness-tm-for-your-ruby-project Continuous Integration Goodness(TM) for Your Ruby Project] (Nick Sieger)
** [https://github.com/nicksieger/ci_reporter/ ci-reporter] (Nick Sieger)
*** [http://caldersphere.rubyforge.org/ci_reporter/ ci_reporter doc]

== CI Implementations  ==

=== Bamboo ===
http://www.atlassian.com/software/bamboo/
Developer Source (proprietary, but you get source code with purchase)

Confirmed to work with JRuby

==== Basic Configuration ====
* Create a "Builder: 
** Label: "JRuby"
** Type: Custom Command  	
** Path: your $JRUBY_HOME
* Builder Configuration
** Argument: $JRUBY_HOME/bin/rake test

==== Blog Posts ====
* [http://furiouspurpose.blogspot.com/2007/07/ruby-on-rails-continuously-integrated.html Ruby on Rails continuously integrated with Bamboo] (Geoffrey Wiseman)
* [http://furiouspurpose.blogspot.com/2007/07/bamboo-ruby-on-rails-test-reports-and.html Bamboo, Ruby on Rails, Test Reports and RCov] (Geoffrey Wiseman)

=== Cruisecontrol ===
http://cruisecontrol.sourceforge.net/
FOSS

=== Cruisecontrol.rb ===
http://cruisecontrolrb.thoughtworks.com/
FOSS

=== Hudson ===

http://hudson-ci.org/

FOSS

Confirmed to work with JRuby

See instructions in [http://blog.huikau.com/2008/01/09/jruby-ruby-continuous-integration-with-hudson/ JRuby / Ruby Continuous Integration with Hudson] to configure.

It also can execute Rake tasks as build steps. See instructions to configure in [http://thinkincode.net/2008/5/22/using-hudson-as-rails-ci-server Using Hudson as Rails CI server]

== Feature Ideas ==
Both Sun and Atlassian have asked what features the Ruby and JRuby community would like to see in CI tools. Add ideas here.

- I'd like Hudson to either support rake and ci:reporter out of the box, or for their to be a plugin to do this.  Allowing configuration in much the same way as you configure an ant project.

== See Also ==
* [http://rossniemi.files.wordpress.com/2007/03/ci-ruby-on-rails-1_0.pdf Continuous Integration Matrix for Ruby on Rails (neglects some viable Java based options)]
* [http://www.carbonfive.com/community/archives/2007/09/ Ruby on Rails Project Report - mentions CI] (Carbon Five)
