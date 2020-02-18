JRuby release preparation
=========================

* Make sure a release has no blockers (tests in later steps are good at shagging issues out)
* Sanity tests (can be done after mvn deploy step but you might be forcing some tags if not ready)
    * Simple rails app with migration(s)
    * unpack bin dist and make sure it works
       * look for unwanted files
       * look for it being too big
    * unpack src dist and compile it
    * Verify windows installers
    * Does jruby-complete work with rubygems (especially on windows)
    * email good env testers (Ben, Terrence)
* export MAVEN_OPTS=-XX:MaxPermSize=768m
* update VERSION
* ./mvnw
* commit and `tag -a` with release notes 
* cd .. && rm -rf release && git clone jruby release && cd release
* mvn clean deploy -Psonatype-oss-release -Prelease
* jrake post_process_artifacts (puts s3 stuff in release subdir)
* push to s3
* gem push jruby-jar-1.7.9.gem
* close and release snapshot repo on sonatype (https://oss.sonatype.org - snapshot repos)
* (broken currently) `jruby -S applet:dist`
    * upload resulting jruby-complete-signedjar to http://jruby.org.s3.amazonaws.com/tryjruby/jruby-complete-signed.jar
* `git push --tags` to propagate tag once release artifacts are out
* Add news item to www.jruby.org page 
    * edit config.yml
    * edit www/downloads.html
    * add www/_posts/2013-12-06-jruby-1-7-9.markdown
    * JRUBY_VERSION=1.7.9 rake issues to get issues list
    * ```rake index``` to update our download index files
    * ```rake server``` and check all links 

* Send emails to ruby-talk@ruby-lang.org, jruby@ruby-lang.org
* tweet
* close milestone on GH and open new milestone for next release
* /msg chanserv OP #jruby <nick>
* /msg chanserv TOPIC #jruby Get 9.1.18.0! http://jruby.org/ | http://wiki.jruby.org | http://logs.jruby.org/jruby/ | http://bugs.jruby.org | Paste at http://gist.github.com
* Send out to all known packagers [[JRubyDistributions]]
   * Change URLs to have SHA256 after file url#SHA256
   * Send PR to https://github.com/sstephenson/ruby-build with PR like: https://github.com/olleolleolle/ruby-build/commit/3b14690ddec4f90cbc264c54af16d617f845c94a
   * Send PR to https://github.com/postmodern/ruby-versions with file checksums and version numbers as detailed here: https://github.com/postmodern/ruby-versions/blob/master/CONTRIBUTING.md
