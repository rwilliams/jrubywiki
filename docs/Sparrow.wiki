'''Sparrow''' (http://github.com/leandrosilva/sparrow) is a very simple JMS client for messaging application based on JRuby.

== Install ==


Add Github as a source for your RubyGem:

<pre>
gem sources -a http://gems.github.com
</pre>

Make install:

<pre>
sudo gem install leandrosilva-sparrow
</pre>

== Example ==

''WARNING: OC4J will be used as Java EE application server, but any other could be used with no problems.''

Create a file named ''my_sparrow_test.rb'' and put the script below:

<pre>
require 'rubygems'  
require 'sparrow'  

# Initial setup (that has to be done once)
jms_client = Sparrow::JMS::Connection::Client.new do |props|  
  props.client_jar_file         = '/oc4j_extended_101330/j2ee/home/oc4jclient.jar'  
  props.initial_context_factory = 'oracle.j2ee.naming.ApplicationClientInitialContextFactory'  
  props.provider_url            = 'ormi://localhost:23791'  
  props.security_principal      = 'oc4jadmin'  
  props.security_credentials    = 'welcome'  
end  

jms_client.enable_connection_factories(  
    :queue_connection_factory => 'jms/MyQueueCF'  
  )  
  
jms_client.enable_queues(  
    :my_queue => 'jms/MyQueue'  
  )  
  
# Send message
jms_client.queue_sender(:my_queue).send_text_message('sparrow rocks!') do |msg|  
  msg.set_string_property('recipient', 'sparrow-example')  
end  
  
# Receive message
jms_client.queue_receiver(:my_queue).receive_message(  
    :timeout  => 5000,  
    :selector => "recipient = 'sparrow-example'"  
  ) do |msg|  
  
  puts msg.is_text_message?    # true  
  puts msg.text                # sparrow rocks!  
end
</pre>

So, now start the OC4J server, create the connection factory (jms/MyQueueCF) and queue (jms/MyQueue), and run above script:

<pre>
jruby my_sparrow_test.rb
</pre>

Output:

<pre>
sparrow rocks! =)
</pre>

== Related Articles ==

* [http://leandrosilva.com.br/2008/08/14/executar-jruby-a-partir-do-java Executar JRuby a partir do Java (pt_BR)] Article about integration of Java with JRuby, showing how running JRuby code from Java.
