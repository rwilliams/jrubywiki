Trying to optimize Jruby in one of my Tomcat Apps sever. I have a timer prog thru which i can monitor resp time in my logs. Looking for an advice on profiling critical JRuby hotspots code on which i can put my timer prog. Like which JRuby packages/classes and ()s should i put my timer on. ? I would appreciate some high level advice pls

My scope of profiling is as follows. Should I add more packages /classes besides the following ? If yes which ones??!!

"org.jruby.runtime.load" 
"org.jruby.rack.RackApplication" 
"jruby.ext.jruby.JRubyFiberLocal"
"org.jruby.runtime.CallSite"