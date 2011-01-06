=JRuby Nailgun Support=

JRuby 1.3.0 and later support execution with [http://martiansoftware.com/nailgun/index.html Nailgun]. Nailgun allows you to start up a single JVM as a background server process and toss it commands from a fast native client almost eliminating JRuby's annoyingly slow start up time. To use Nailgun with JRuby:

# Download JRuby.
# Expand the JRuby archive (source or binary) and change to the distribution directory.
# If you are running on UNIX-like OS, build the nailgun server with either:
## <code>ant build-ng</code>
## <code>cd tool/nailgun; configure; make</code>
## <code>cd tool/nailgun; make</code> (JRuby 1.3.x only)
# Start the JRuby nailgun server in the background: <code>jruby --ng-server &</code>
# You can now run JRuby commands through that server using the <code>jruby --ng</code> command instead of <code>jruby</code>

For more information, read [http://blog.headius.com/2009/05/jruby-nailgun-support-in-130.html]

==Note on development with Nailgun==
If you rebuild JRuby, be sure to restart the nailgun server; or else the client will get incorrect results.