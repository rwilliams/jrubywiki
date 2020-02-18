= File Column Rails Plugin Notes =

NOTE: This note applies to Rails Integration 1.0 and JRuby 0.9.8.

1. This plugin inadvertently calls ARGV (due to "require 'testcase'"). RailsIntegration way of embedding JRuby doesn't init the constant ARGV. Just add ARGV = [] at the beginning of your environment.rb as a workaround.

2. Process.pid method is called by this plugin which is not implemented in JRuby at this time. Search for this reference and remove it as it is pretty safe since in a WAR deployment there is generally likely to be only one instance.
