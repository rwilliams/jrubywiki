This page shows an example of using the Java JMX packages directly from Ruby, but there are two gems which exist to make using JMX must less painful.  Consider using these packages before you break down and code this stuff by hand:

* [[http://www.kenai.com/projects/jmxjr]]
* [[http://github.com/jmesnil/jmx4r/]]

This code snippet shows how to access ActiveMQ's Dead Letter Queue (DLQ):

```ruby
 require 'java'
 
 import java.rmi.MarshalledObject
 import java.rmi.registry.LocateRegistry
 import javax.management.ObjectName
 
 # Connect to the JNDI lookup server
 registry = LocateRegistry.get_registry("localhost", 1099) 
 
 # List available RMI objects
 names = registry.list
 puts 'Available RMI objects:'
 p names.to_a
 puts
 
 # Fetch a remote proxy to the JMX RMI object
 remote = registry.lookup("jmxrmi")
 puts "Got remote JMX object: #{remote.inspect}"
 con = remote.new_client nil
 puts "Got connection: #{con.inspect}"
 
 # List JMX domains
 domains = con.get_domains nil
 puts "Found domains:"
 p domains.to_a
 
 # List available MBeans
 mbeans = con.query_names nil, nil, nil
 puts 'Available MBeans:'
 mbean_names = mbeans.to_a.map{|n| n.to_s}.sort
 puts mbean_names.join("\n")
 puts
 puts
 
 
 # Find the dead letter queue in ActiveMQ 
 queue = mbean_names.find{|n| n =~ /org.apache.activemq.*Type=Queue.*Destination=ActiveMQ.DLQ/}
 if queue
   puts 'Found ActiveMQ DLQ:'
   p queue
 
   puts "Purging DLQ"
   name = ObjectName.new(queue)
   res = con.invoke name, 'purge', MarshalledObject.new(nil), nil, nil
   puts "Result from purge: #{res.inspect}"
 end
```

