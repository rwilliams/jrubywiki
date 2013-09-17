Unlimited Strength Crypto
=========================

In order to comply with US cryptography export laws, JDKs and JREs based on OpenJDK will ship -- but not enable -- unlimited-strength cryptography. In order to enable unlimited-strength crypto, you have two options:

Install the "Unlimited Strength" policy files from Oracle
---------------------------------------------------------

These files enable unlimited strength crypto. Download the "Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files" from [Oracle's JavaSE Downloads](http://www.oracle.com/technetwork/java/javase/downloads/index.html) page and follow the README.txt instructions contained therein.

Disable the crypto restriction programmatically
-----------------------------------------------

(This may apply to Java 7+ only)

The policy files above are loaded once at startup and used to set an "isRestricted" field on the javax.crypto.JceSecurity class. Using JRuby's Java integration, this field can be modified.

Putting the following code somewhere in your boot process before unlimited-strength crypto is needed will allow that crypto to succeed:

```ruby
security_class = java.lang.Class.for_name('javax.crypto.JceSecurity')
restricted_field = security_class.get_declared_field('isRestricted')
restricted_field.accessible = true
restricted_field.set nil, false
```

Or as a one-liner:

```ruby
java.lang.Class.for_name('javax.crypto.JceSecurity').get_declared_field('isRestricted').tap{|f| f.accessible = true; f.set nil, false}
```

Note that this circumvents the retrictions intended to comply with US export laws, and you may be in violation of those laws if you are from a country in which the US government intends to restrict access to cryptography technologies.

See also the Wikipedia article on [Export of cryptography in the United States](http://en.wikipedia.org/wiki/Export_of_cryptography_in_the_United_States).