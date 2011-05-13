JRuby versions prior to 1.6 did not support Ruby C extensions, and even in 1.6 the support is still "in development" and considered experimental. This page lists common C extensions and non-C alternatives you can use to replace them.

* **[RDiscount][]** - use [Maruku][] (pure Ruby) or [markdown_j][] (wrapper around a Java library)

* **[RMagick][]** - try [RMagick4J][] (implements ImageMagick functionality in Java) or preferably use alternatives [mini_magick][] & [quick_magick][]. For simple resizing, cropping, greyscaling, etc look at [image_voodoo][].

* **[Unicorn][]** - try any one of the following [[JRuby-based servers|Servers]]: [Trinidad][], [Mizuno][], [Kirk][] or [TorqueBox][].


[RDiscount]: https://github.com/rtomayko/rdiscount
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
