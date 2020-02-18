This page will follow a project developing an airport fuelling system.  We will try to describe the different components used, and some of the pitfalls detected, but more importantly focus on working examples and best practises.

EVERYONE is allowed to comment, add or edit to this page!  The more views the better!

__TOC__

==Purpose==

AIFUDIS offers information about flights at one airport to a Dispatcher controlling the fuel trucks at the airport.  The dispatcher then assigns operators and their trucks to specific flights.  The operators then fuel the aircrafts and report progress and the finished transaction back to the central system and the dispatcher.

At the end of the day reports on all transactions and the state of the trucks and fuel tanks are used to send invoices and order more fuel.

==Architecture==

===Dispatcher===

* Eclipse RCP 3.4
* Java 6
* JRuby 1.1.2 in selected parts.
* Firefox 3
* Spring 2.0

===Operator===

* Eclipse RCP 3.4
* Java 6
* JRuby 1.1.2 in selected parts.

===Administration===

* Ruby on Rails 2.1
* JRuby 1.1.2
* Java 6
* PostgreSQL 8,3

===Communicator===

* Ruby on Rails 2.1
* JRuby 1.1.2
* Java 6
* ActiveMessaging Plugin
* ActiveMQ 5.1
* PostgreSQL 8,3

===Flight Importer===

Polls flights from an Oracle 10 database using ActiveRecord.

* Ruby on Rails 2.1
* JRuby 1.1.2
* Java 6
* ActiveMessaging Plugin
* ActiveMQ 5.1
* PostgreSQL 8,3


==Progress==

===0.0.0 2008-05-15 Infrastructure===

* All the main components of the system are up and running with no functionality.  That means all infrastructure is in place.  Woohoo!


===0.1.0 2008-06-16 Basic models, Working Flight List, Administration of base data.===

* The release went well and the customer is happy!  Below is a screen shot of the flight list.

[[image: Dispatcher_0.1.0.jpg]]

Currently, JRuby is used to fetch data from an external Oracle database, post flight updates to an ActiveMQ topic, handle all administration using a Rails application, and to unmarshal a YAML stream of flight updates on the dispatcher.


===0.2.0 2008-08-03 First operator functions===

* In progress...

==Using JRuby in an Eclipse RCP Plugin==

===Activators, Editors, views, etc.===

When defining plugin activators, editors, views etc., you do this in the plugin.xml file.  The classes mentioned here are loaded using reflection.  We have not found a way to load ruby classes this way, so all these classes need to be Java classes.  This does continually push development towards using Java instead of Ruby.

We would like a way to use Ruby classes directly as Editors, Views, etc.  Please comment if you have an idea how to do this.

''Comment from DominikM''
I don't think it would help with activators but for everything that works with the ExtensionRegistry there is a way of getting external non-java implementations to work: IExecutableExtension

See: http://help.eclipse.org/stable/index.jsp?topic=/org.eclipse.platform.doc.isv/reference/api/org/eclipse/core/runtime/IExecutableExtension.html

By implementing something like a JRubyExtensionAdapter which is used in extension points like this: xxx.yyy.JRubyExtensionAdapter:/scripts/views/MyRubyView.rb you could get what you want with only a little generic java code.

Everything that is put after the double-colon is provided to the IExecutableExtension via setInitializationData(). Hope that helps :) Good luck on your project and please continue to update your progress.

==Organizing ruby code.==

The Eclipse RCP SDK gives good guidelines on how to organize your Java code.  A similar guide is needed for organizing the Ruby code.  A project specific guide will emerge, but we would like to see if this could be used across projects.

We have a few ideas on where to put the Ruby code:

* Separate /rubysrc folder.
* A /src/ruby folder.
* /lib
* Together with the Java code that uses the Ruby code.  If my.domain.JavaClass uses file my_ruby_class.rb, the the my_ruby_class.rb file id placed together with the JavaClass.java file in the /src/my/domain folder.  This enables the Java class to load the Ruby class using Classloader.getResourceAsStream().


If you have any experience with organizing ruby code in a Java project, please share here!


==Sending objects over the network==

We use ActiveMQ with STOMP as transport for messaging, supported by the excellent ActiveMessaging plugin.

Here is an example of how we send objects over the network.  Only the object fields are sent, no code, and we use YAML for this since it is simple, readable, and has parsers on both Java and Ruby platforms.  *flight* is an ActiveRecord model.

===JRuby sender===

  require 'activemessaging/processor'
  
  class FlightUpdater
    include ActiveMessaging::MessageSender
    extend ActiveMessaging::MessageSender
  
    QUEUE = :flight_updates
  
    publishes_to QUEUE
  
    def self.update flight
      payload = YAML.dump flight.attributes
      publish(QUEUE, payload)
    end
  
  end


===Java receiver===

  ...
  jruby.defineReadonlyVariable("$flight", JavaEmbedUtils.javaToRuby(jruby, flight));
  Object map_as_ruby_object = jruby.evalScriptlet("YAML.load($flight)");
  @SuppressWarnings("unchecked")
  Map<Object, Object> flight_attributes = 
          (Map<Object, Object>) JavaEmbedUtils.rubyToJava(jruby, (org.jruby.runtime.builtin.IRubyObject) map_as_ruby_object, Map.class);
  ...

This is a bit more complicated than necessary, but we wanted to try the Ruby-To-Java conversion mechanism.

Please leave a comment if you would like more details.
