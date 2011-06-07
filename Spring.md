There isn't much documentation about accessing JRuby from within Java, everyone seems to do things the other way around. However let's say you've a nice java web app or something similar and you're looking enviously at all those Rubyists wondering if these folks are getting the jump on you. Wouldn't it be great if we were able to simply replace little chunks of our java application with Ruby. Wouldn't it be a wonderful thing if we were able to delegate bits of our statically typed, rigid web application to a scripting language. You probably already do some of this with Velocity or Freemarker but those are typically just cosmetics, what you want is to be able to **dynamically edit business logic** on the fly. There are a few business rules engines out there, there's Droolz (now JBoss Java Rules) and various open and closed source BPM types of solutions. Nothing really has the buzz that Ruby has.

The spring docs actually miss the point of integration between Java and JRuby (I'm sure someone will sort that out), but in the meantime here's a neat way to integrate both technologies without creating dependencies on either side. **It's cool!**

First you'll need this in your context xml. Mine is simply applicationContext-ruby.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:lang="http://www.springframework.org/schema/lang"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
	http://www.springframework.org/schema/lang
	http://www.springframework.org/schema/lang/spring-lang-2.0.xsd">

	<lang:jruby id="messageService"
		script-interfaces="spike.Messenger"
		script-source="classpath:RubyMessenger.rb">
		<lang:property name="message" value="Hello World!"/>
	</lang:jruby>

</beans>
```

What I'm asking Spring to do here is to wire up something found in the `RubyMessenger.rb` file, to my interface `spike.Messenger` and return me an instance of a Messenger. That instance will have a property set on it, by Spring. Take a look at my Messenger specification.

```java
package spike;

public interface Messenger {
    public String getMessage();
}
```

The main thing to note here is that spring is setting the property in my bean, however from the Java side of things, there is no setter, nor constructor available; the values are injected via JRuby. Java has no idea that the implementation of Messenger is coming from Ruby. Let's look at that implementation:

```ruby
class RubyMessenger
	def setMessage(m)
		@@message = m
	end

	def getMessage
		@@message
	end
end

RubyMessenger.new
```

The last line just avoids a bit of stress on the part of Spring by giving it an object to inspect and work with.

The most special thing about RubyMessenger.rb is that it is not special at all. It doesn't know that it's actually being injected into a Java framework. There's no `require 'java'` in there, it's pure, straight, unencumbered Ruby. Ruby that could run in a normal Ruby shell.

Now isn't that just so cool? If you're integrating into an existing Java app, you can continue running all your unit tests and refactor classes into a completely different language. You can steal huge chunks of already written ruby libraries and applications and using a simple facade pattern, have those chunks of system working for you with no changes, no knowledge of the behind the scenes Java.

Here's a snippet of code that executes the sample app above. To run it you'll need the spring.jar (v2.0.7), JRuby.jar (v1.0.1) and cglib-nodep.jar (v2.1_3):

```java
package spike;

import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class RubyRunner {
	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext(
						new String[]{"applicationContext-ruby.xml"});
		BeanFactory factory = (BeanFactory) context;

		Messenger m = (Messenger) factory.getBean("messageService");
		System.out.println(m.getMessage());
	}
}
```

I'm actually delegating the functionality from my Java app where we load and manipulate a lot of huge text files. This is perfect because with a dynamic language behind the scenes we can patch up the production systems to deal with anomalies without re-deploying an entire website.

Now what would be nice is for Spring to automatically map ruby accessors to Javabeans. Using the above example, Spring would bind `setMessage` to `message=()` and `getMessage` to the Ruby method `message`. In fact I think that would be so useful, I'm going to head over and ask the nice people as springframework.org to do just that.
