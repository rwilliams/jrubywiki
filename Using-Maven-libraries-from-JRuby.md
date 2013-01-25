JRuby's version of rubygems has support for treating Maven dependencies as if they are gems. Just `gem install mvn:[groupId]:[artifactId]` and then `require 'mvn:[groupId]:[artifactId]'` in your ruby code.

An example:

gem install mvn:com.yammer.metrics:metrics-core

```ruby
require 'java'
require 'rubygems'
require 'mvn:com.yammer.metrics:metrics-core'

counter = com.yammer.metrics.Metrics.new_counter(...)
```