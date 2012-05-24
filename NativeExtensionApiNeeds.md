For JRuby.x we want to make native extensions work well against a stable API.  Our current native extension API is not bad, but it hits our internals too much and the risk of breaking is too large with a major release.  Additionally, a stable isolated API will also provide a better way to document how to make native extensions.

# Use-cases

* Defining modules/classes:

```java
 RubyModule mHitimes = runtime.defineModule("Hitimes");
 RubyClass  cHitimesError  = mHitimes.defineClassUnder("Error", cStandardError, cStandardError.getAllocator());
```

* Accessing constants:

```java
 RubyClass errorClass = runtime.fastGetModule("Hitimes").fastGetClass( "Error" );
```


* Defining methods:

```java
 cHitimesInterval.defineAnnotatedMethods( HitimesInterval.class );
```

I suspect this usage is perfectly fine.  Perhaps we should put an annotation for stable native extension APIs.


