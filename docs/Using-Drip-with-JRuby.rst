## Overview

[drip](https://github.com/ninjudd/drip) is a command-line tool that can be used to dramatically lower perceived JVM startup time. It does this by preloading an entirely new JVM process/instance in the background and allowing you to simply use the preloaded environment. This can improve startup of JRuby-based applications significantly.


## Install Drip
Install drip if you haven't already (see https://github.com/ninjudd/drip)

You can check out drip directly and build it, or use Homebrew on OS X.

```bash
brew update && brew install drip
```

## Environment Setup

JRuby uses the `JAVACMD` environment variable (if present) as its executable (usually `which java`). 
drip uses the `DRIP_INIT_CLASS` environment variable to determine the main class to load. JRuby has a native Java class already setup for this purpose: [org.jruby.main.DripMain](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/main/DripMain.java).

```bash
export JAVACMD=`which drip`
export DRIP_INIT_CLASS=org.jruby.main.DripMain
```

On Drip versions earlier than [820f869](https://github.com/ninjudd/drip/commit/820f8696187c4ac998e54b1ee4d8a2fdc281d1ed) `DRIP_INIT` must be set.
```bash
export DRIP_INIT=""
```

## Project Setup

Put any project-specific initialization code (Ruby code) in `dripmain.rb` in the current working directory. This file is automatically called by the special `org.jruby.main.DripMain` class when initializing the standby JVM process.

Rails Example:

```bash
# Rails project:
require_relative 'config/application'

# non-Rails Bundler-controlled project
require 'bundler/setup'
Bundler.require
```

## rvm integration

If you would like to use drip automatically whenever you switch to JRuby with `rvm`, you will need to add a new hook file at `$rvm_path/hooks/after_use_jruby_drip` with the following content:

```bash
#!/usr/bin/env bash

if [[ "${rvm_ruby_string}" =~ "jruby" ]]
then
  export JAVACMD=`which drip`
  export DRIP_INIT_CLASS=org.jruby.main.DripMain
  
  # settings from: https://github.com/jruby/jruby/wiki/Improving-startup-time
  export JRUBY_OPTS="-J-XX:+TieredCompilation -J-XX:TieredStopAtLevel=1 -J-noverify" 
fi
```

Then you'll need to make that file executable:

```bash
chmod +x $rvm_path/hooks/after_use_jruby_drip
```

See also:

* [Drip Experiments (testing by Charles Nutter)](https://gist.github.com/4156388)
* [JRuby Drip Initial Class](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/main/DripMain.java)
* [ninjudd's drip README](https://github.com/ninjudd/drip)
