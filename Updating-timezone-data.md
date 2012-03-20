If you find that a JRuby release does not have the latest time zone data, and if you think this is important, you can build JRuby from source with the most up-to-date [Olson database](http://www.iana.org/time-zones).

## Prerequisites
Environment in which [`ftp` Ant task](http://ant.apache.org/manual/Tasks/ftp.html) can run. This means that you have [Apache Commons Net](http://commons.apache.org/net/index.html) installed and available for your JVM (and `ant`).

## Steps
1. Move to the root of the JRuby source.
1. Optionally edit `default.build.properties` and set `tzdata.latest.version` to the version you want. (If you don't know what you need, you can leave the default in. We try to keep this value fairly up-to-date.)
1. Run `ant clean compile-tzdata jar`.
1. Confirm that you have the new version of tzdata:
```
./bin/jruby -rrbconfig -e 'p RbConfig::CONFIG["tzdata.version"]'
"2012b"
```