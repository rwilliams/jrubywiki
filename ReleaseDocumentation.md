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
    * email good env testers (Ben, Terrence)
* export MAVEN_OPTS=-XX:MaxPermSize=768m
* mvn versions:set -DnewVersion=1.7.9 -Pall
* commit and tag
* cd .. && rm -rf release && git clone jruby release && cd release
* mvn clean deploy -Psonatype-oss-release -Prelease
* jrake post_process_artifacts (puts s3 stuff in release subdir)
* push to s3
* gem push jruby-jar-1.7.9.gem
* close and release snapshot repo on sonatype (https://oss.sonatype.org - snapshot repos)
* `jruby -S applet:dist`
    * upload resulting jruby-complete-signedjar to http://jruby.org.s3.amazonaws.com/tryjruby/jruby-complete-signed.jar
* Add news item to www.jruby.org page 
    * edit config.yml
    * edit www/downloads.html
    * add www/_posts/2013-12-06-jruby-1-7-9.markdown
        * JRUBY_VERSION=1.7.9 rake issues to get issues list
        * rake index to update our download index files
    * LANG="en_US.UTF-8" LC_ALL="en_US.UTF-8" mri19 -S rake deploy
    * LANG="en_US.UTF-8" LC_ALL="en_US.UTF-8" mri19 -S rake deploy

* Send emails to dev@jruby.codehaus.org, user@jruby.codehaus.org, ruby-talk@ruby-lang.org
* tweet
* close milestone on GH and open new milestone for next release
* Update wikipedia entry
* /msg chanserv OP #jruby <nick>
* /msg chanserv TOPIC #jruby Get 1.7.16! http://jruby.org/ | http://wiki.jruby.org | http://logs.jruby.org/jruby/ | http://bugs.jruby.org | Paste at http://gist.github.com
* Send out to all known packagers [[JRubyDistributions]]
