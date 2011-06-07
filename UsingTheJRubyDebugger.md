There is a Java-based JRuby implementation of the fast ruby debugger rdebug.

See: [JRuby Fast Debugger](http://debug-commons.rubyforge.org/#jruby-debug)

Note that more recent versions of jruby (1.5+) already include the gem pre-bundled, so a

```ruby
require 'rubygems'
require 'ruby-debug'
debugger
```

is probably all you'll need.

If you're running an older version of jruby then you'll need to follow these instructions:

* Manually download the most recent ruby-debug-base-0.10.3.x-java.gem from [http://rubyforge.org/frs/?group_id=3085 debug-commons].
* Install the Gem into your JRuby Gem repository:
```bash
jruby -S gem install ruby-debug-base-0.10.3.2-java.gem
```
* **Leave the directory or remove ruby-debug-base-0.10.3.2-java.gem from the directory** and install ruby-debug with:
```bash
jruby -S gem install --ignore-dependencies ruby-debug
jruby -S gem install --ignore-dependencies ruby-debug-ide # needed only for IDEs
```
* Install the columnize gem (unless you already have):
```bash
jruby -S gem install columnize
```

**Note**: the `--debug` option used below works only with JRuby 1.1.3 and later. For older versions you need to stick with the old option, using `-J-Djruby.reflection=true -J-Djruby.compile.mode=OFF` instead `--debug`.

Starting the debugger on a Rails application:

```bash
cd my_rails_app
jruby --debug -S rdebug script/server
```

Starting the debugger to test a local ruby program and adding ''lib/'' to the load path:

```bash
ruby --debug -S rdebug -Ilib rubyprogram.rb 
```

or you can call it programmatically within the script

```ruby
require 'ruby-debug'
debugger
```

just make sure to run it under a jruby with the --debug flags passed to it, or "next" will always act as if it were  the "step" command.

External Resources:

* [trunk jruby-debug README](http://debug-commons.rubyforge.org/svn/jruby-debug/trunk/README)
* [Debugging with ruby-debug](http://bashdb.sourceforge.net/ruby-debug.html)
* [JRuby Debugger Troubleshooting - Forum Thread](http://www.intellij.net/forums/thread.jspa?messageID=5225735)
