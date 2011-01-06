Many Ruby libraries are packaged as gems.  Unfortunately many of these gems assume that they will only be used in an environment that has Rubygems installed.  This is a problem if you want to distribute an applications as a jar file that uses unpacked gems.  To get around this, one solution is to stub out the Rubygems classes.  For example, you could have a file in your application named rubygems.rb that contains the class and method Rubygems code will normally call.

<pre>
Object.send(:remove_const, :Gem) if Object.const_defined? :Gem
Kernel.send(:remove_method, :gem) if Kernel.respond_to? :gem

module Gem
end

module Kernel
  def gem *args; end
end
</pre>

Assuming your rubygems.rb file is located first on the classpath, when any library expecting Rubygems to be installed calls "require 'rubygems'" they will get you stubbed out version.  Since there is no automatic load path modification from Rubygems itself you will need to ensure your unpacked gems are added to the load path.

Also, this will interfere with any utilities you are using that relies on Rubygems.  This includes things such as the debugger.
