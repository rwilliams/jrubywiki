RCov has nascent support for JRuby, but it is not yet in a release. To install it, follow these instructions (at least until there's a -jruby platform gem for rcov proper):

 git clone git://github.com/relevance/rcov.git
 cd rcov
 jruby -S rake build rcov.gemspec
 cd pkg
 jruby -S gem install rcov-0.8.3.2-java.gem
