Nailgun is one way to tune Java processes - the idea is to push the startup overhead off to a central process, keeping the clients quick and thin.

http://www.martiansoftware.com/nailgun/background.html

The original guide to using it with JRuby is here:

http://blog.headius.com/2009/05/jruby-nailgun-support-in-130.html

As of JRuby 1.6.3 (at least when installed via RVM), it is turned on by default.

This is done via an RVM hook, see $RVM_HOME/hooks/after_use_jruby.

To turn off this feature, delete this file.  (this is probably not the official way to disable hooks, but I am afraid I cannot find that, yet...)