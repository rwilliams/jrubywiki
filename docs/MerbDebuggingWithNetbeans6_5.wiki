To make able to run Netbeans with Merb application debugging i've used:
JRuby 1.1.5 (from git trunk from 4'th of December 2oo8, but seems that official 1.1.5 will work here too)
Netbeans 6.5

First we need to install ruby-debug-ide for JRuby. There's info how to install this gem here: http://wiki.netbeans.org/RubyDebugging#section-RubyDebugging-JRuby

After we have installed those gems, it's time for little hack in out Merb application:
in Application ROOT do the following:

 <nowiki>mkdir script</nowiki>

 <nowiki>touch server</nowiki>

and put there this content:


 <nowiki>
 #!/usr/bin/env ruby
 require 'rubygems'
 version = ">= 0"
 if ARGV.first =~ /^_(.*)_$/ and Gem::Version.correct? $1 then
 version = $1
 ARGV.shift
 end
 gem 'merb-core', version
 load 'merb'
 </nowiki>


Note: that it's working properly with Mongrel server only. It seems there's a bug in Glassfish delivered with Netbeans 6.5 and this application server wont work with debugger. (in my case)

Note: It does not work with newer Merb version which is executed in different way than before. Workaround needed!
