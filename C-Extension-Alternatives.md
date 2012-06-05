JRuby versions prior to 1.6 did not support Ruby C extensions, and even in 1.6 the support is still "in development" and considered experimental. This page lists common C extensions and non-C alternatives you can use to replace them.

* **[RDiscount][]** - Use [kramdown][], [Maruku][] (pure Ruby) or [markdown_j][] (wrapper around a Java library)

* **[RMagick][]** - Try [RMagick4J][] (implements ImageMagick functionality in Java) or preferably use alternatives [mini_magick][] & [quick_magick][]. For simple resizing, cropping, greyscaling, etc look at [image_voodoo][].

* **[Unicorn][]** - Try any one of the following [[JRuby-based servers|Servers]]: [Trinidad][], [Mizuno][], [Kirk][], [TorqueBox][] or [Puma][].

* **[Thin][]** - Thin might compile and run but is not recommended. Try any one of the following [[JRuby-based servers|Servers]]: [Trinidad][], [Mizuno][], [Kirk][], [TorqueBox][] or [Puma][].

* **[Typhoeus][]** - The C extension [doesn't currently compile](https://github.com/dbalatero/typhoeus/issues/65). There's no equivalent library for JRuby, but you might try any of the pure-Ruby HTTP clients (`net/http`, httpclient, etc.). There are also several Java HTTP client libraries that will work ([Apache HttpClient][], [HttpURLConnection][], etc).

* **mysql** - Use [activerecord-jdbc-adapter][] instead along with `jdbc-mysql`.

* **mysql2** - Use [activerecord-jdbc-adapter][] instead along with `jdbc-mysql`.

* **sqlite3** - Use [activerecord-jdbc-adapter][] instead along with `jdbc-sqlite3`.

* **[Nokogiri][]** - For best results, use the pure-Java version of Nokogiri (default after v1.5).

* **[yajl-ruby][]** - Try `json` or `json_pure` instead. Unfortunately there is no known equivalent JSON stream parser.

* **[bson_ext][]** - `bson_ext` isn't used with JRuby. Instead, some native Java extensions are bundled with the `bson` gem.

* **[win32ole][]** - Use the `jruby-win32ole` gem (preinstalled in JRuby's Windows installer).

* **[curb][]** - [Rurl][] is an example how to implement _some_ of curb's functionality using [Apache HttpClient][]

* **[therubyracer][]** - Try using [therubyrhino][] instead.

Please add to this list with your findings.

*Note that the [JRuby-Lint][] gem parses the contents of the list above to use for its Ruby gem checker. In order for JRuby-Lint to use the information, please adhere to the `gem_name - instructions` format.*

[RDiscount]: https://github.com/rtomayko/rdiscount
[kramdown]: http://kramdown.rubyforge.org
[Maruku]:https://github.com/nex3/maruku
[markdown_j]: https://github.com/nate/markdown_j
[RMagick]: https://github.com/rmagick/rmagick
[RMagick4J]: https://github.com/Serabe/RMagick4J
[mini_magick]: https://github.com/probablycorey/mini_magick
[quick_magick]: https://github.com/aseldawy/quick_magick
[image_voodoo]: https://github.com/jruby/image_voodoo
[Unicorn]: http://unicorn.bogomips.org/
[Trinidad]: https://github.com/trinidad/trinidad
[Mizuno]: https://github.com/matadon/mizuno
[Kirk]: https://github.com/strobecorp/kirk
[TorqueBox]: http://torquebox.org/
[Puma]: http://puma.io/
[Thin]: http://code.macournoyer.com/thin/
[Typhoeus]: https://github.com/dbalatero/typhoeus
[activerecord-jdbc-adapter]: https://github.com/nicksieger/activerecord-jdbc-adapter
[JRuby-Lint]: https://github.com/jruby/jruby-lint
[Nokogiri]: http://nokogiri.org/
[yajl-ruby]: https://github.com/brianmario/yajl-ruby
[bson_ext]: https://github.com/mongodb/mongo-ruby-driver
[Apache HttpClient]: http://hc.apache.org/httpcomponents-client-ga/
[HttpURLConnection]: http://download.oracle.com/javase/1,5.0/docs/api/java/net/HttpURLConnection.html
[win32ole]: http://www.ruby-doc.org/stdlib/libdoc/win32ole/rdoc/index.html
[Rurl]: https://github.com/rcyrus/Rurl
[curb]: http://curb.rubyforge.org/
[therubyracer]: https://github.com/cowboyd/therubyracer
[therubyrhino]: https://github.com/cowboyd/therubyrhino