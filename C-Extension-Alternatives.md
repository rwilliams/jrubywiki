JRuby versions prior to 1.6 did not support Ruby C extensions, and even in 1.6 the support is still "in development" and considered experimental. This page lists common C extensions and non-C alternatives you can use to replace them.

* **[[RDiscount|https://github.com/rtomayko/rdiscount]]** - use [[Maruku|https://github.com/nex3/maruku]] (pure Ruby) or [[markdown_j|https://github.com/nate/markdown_j]] (wrapper around a Java lib)

* **[[RMagick|https://github.com/rmagick/rmagick]]** - try [[RMagick4J|https://github.com/Serabe/RMagick4J]] (implements ImageMagick functionality in Java) or preferably use alternatives [[mini_magick|https://github.com/probablycorey/mini_magick]] & [[quick_magick|https://github.com/aseldawy/quick_magick]]