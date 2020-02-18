== Introduction ==
AOT compilation of Ruby on Rails controllers and models can be useful for ''protecting''/''hiding''/''obfuscating''/''obscuring'' your Ruby on Rails source code when you develop applications not hosted by you. (Naturally this is not a secure solution for protecting source code.)

JRuby 1.1 is required.

== Howto ==
* Compile your controller/model using ''jrubyc''
<pre>$jrubyc your_controller.rb</pre>
* Choose one of the following three ways:
:1. Two files, different file prefixes
:*Rename your compiled file to e.g. ''your_controller_compiled.class'' and place it in the same directory as your Ruby controller/model
::<pre>$mv ruby/your_controller.class ./your_controller_compiled.class</pre>
:* Replace the content of your original Ruby controller/model (e.g. ''your_controller.rb'') with a reference to your newly compiled file
::<pre>require 'your_controller_compiled'</pre>
:2.  Two files, same file prefix ''(does this work by luck, or are .class files prioritized?)''
:*Move your compiled file to the same directory as your Ruby controller/model
::<pre>$mv ruby/your_controller.class ./</pre>
:* Replace the content of your original Ruby controller/model (e.g. ''your_controller.rb'') with a reference to your newly compiled file
::<pre>require 'your_controller'</pre>
:3. Replace your .rb file with your .class file
:<pre>$mv ruby/your_controller.class ./your_controller.rb</pre>
