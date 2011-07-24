JRuby release preparation
=========================

* Make sure a release has no blockers (tests in later steps are good at shagging issues out)
* Preliminary testing
    * Try and determine if we break any common extensions (jruby-rack, ar-jdbc, joni, jsr233 impl, ...)
    * Create a test dist file
        * Verify bin works
        * Verify src with 'ant test' and 'ant spec'
        * Install a gem or two
* Create new branch [Only for RC1]
    * grb create jruby-1_5
* [branch] Update default.build.properties to reflect new version [1.5.0.RC1]
* [branch] Update maven poms: `jruby -S rake maven:updatepoms VERSION={newversion}`
* [master] Update default.build.properties to reflect next version [ -> 1.6.0.dev]
* [master] Update maven poms: `jruby -S rake maven:updatepoms VERSION={newversion}`
* Make tag using git tag: `git tag X.Y; git push --tags`
* Check out clean clone of release branch: `cd .. && rm -rf release && git clone jruby release && cd release` (for example)
* Set your java version to Java 5 for building the release.
* If you use rvm then `rvm use system` to prevent [gem accident](http://jira.codehaus.org/browse/JRUBY-5231 JRUBY-5231)
* Run **ant dist** to generate source and binary distribution files, jruby-complete jar, installers, jruby-jars gem, and MD5/SHA checksum files.
* Make `ant test` runs on Linux, MacOS, and Windows with produced source distribution
* Verify common tasks as extra sanity check (Windows + one other system) [with binary distro -- try some with .zip dist and some with .tar.gz dist]:
    * `jruby samples/swing2.rb`
    * `jruby -S jirb`
    * install rails + ar-jdbc
        * setup simple web app and exercise it
        * install and run goldberg (actually any more sophisticated app you know works)
* Upload dist files, jruby-complete.jar, jruby-jars gem and MD5 files to S3: https://jruby.org.s3.amazonaws.com/downloads/{version}
* Release jruby-jars gem: `gem push dist/jruby-jars-{VERSION}.gem`
* `mvn deploy -DupdateReleaseInfo=true` at the root of the project to build and deploy all maven artifacts
* `jruby -S applet:dist`
    * upload resulting jruby-complete-signedjar to http://jruby.org.s3.amazonaws.com/tryjruby/jruby-complete-signed.jar
* Add news item to www.jruby.org page 
* Update MainPage on JRuby wiki pointing to new binary
* Update Ruby Application Archive
* Close all resolved bugs associated with the release in JIRA
* Release the version in JIRA
* Send emails to dev@jruby.codehaus.org, user@jruby.codehaus.org, ruby-talk@ruby-lang.org (plus any thing else you may find useful -- internal lists, etc...)
* Update wikipedia entry
* Send out to all known packagers [[JRubyDistributions]]