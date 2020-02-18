Mentawai (http://www.mentaframework.org/?loc=en) now fully supports JRuby. You are now able to write your actions in Ruby (or your whole application) or you can call any Ruby code from your Mentawai Java code.


To support plain Ruby, all you need to do is copy the jruby.jar file inside the /WEB-INF/lib directory of your Mentawai web application. However, if you plan to use RubyGems and the Ruby Libraries (like 'date' for example) you will need to install JRuby in the machine running your Tomcat web server. To install JRuby is very easy. Follow the steps below:

* Download the zip file with the JRuby installation from here: http://dist.codehaus.org/jruby/ (Ex: jruby-bin-1.1.1.zip)

* Extract everything inside your root directory (or any directory)

* Set the environment variable JRUBY_HOME. (Ex: JRUBY_HOME=c:\jruby-1.1.1)

* Set the environment variable JRUBY_OPTS to -rubygems so that you don't need to require 'rubygems' in your code.

* Change your PATH environment variable to include %JRUBY_HOME%\bin so that you can call the jruby script from anywhere in your shell.


If you did the above steps correctly, you should be able to play with jruby in your shell. Check the examples below: 

 c:\>jruby -v
 ruby 1.8.6 (2008-04-22 rev 6555) [x86-jruby1.1.1]

 c:\>jruby -S gem -v
 1.1.0
<pre>
 c:\>jruby -S gem list --local

 *** LOCAL GEMS ***

 hoe (1.5.1)
 hpricot (0.6)
 jruby-openssl (0.2.3)
 mechanize (0.7.6)
 rake (0.8.1)
 rspec (1.1.3)
 rubyforge (0.4.5)
 sources (0.0.1)
</pre>


Now let's check an example of how to run Ruby inside your Mentawai web application. (download Mentawai 1.13 if you need from here: http://www.mentaframework.org/beta/mentawai.jar)


All your ruby code must be placed inside the /WEB-INF/ruby/ directory of your web application. Mentawai loads (<i>requires</i>) any Ruby file from this directory plus it will reload any modified file pretty much like a JSP.


An action written in Ruby: /WEB-INF/ruby/action_test.rb
<pre>
  include_class 'org.mentawai.core.BaseAction'

  class MyFirstRubyAction < BaseAction
  
    def execute
      output["msg"] = "Hello from JRuby !!!!"
      :success
    end
  
  end
</pre>


A Java Action calling Ruby code: /WEB-INF/src/org/hello/action/Hello.java 

<pre>
package org.hello.action;

import java.util.Iterator;
import java.util.Map;

import org.mentawai.core.BaseAction;
import org.mentawai.jruby.JRubyInterpreter;

public class Hello extends BaseAction {
	
	JRubyInterpreter ruby = JRubyInterpreter.getInstance();
	
	public String execute() {
		
		Object users = ruby.eval("u = Users.new");
		
		Map map = (Map) ruby.eval("u.get_users");
		
		Iterator iter = map.keySet().iterator();
		
		StringBuilder sb = new StringBuilder();
		
		while(iter.hasNext()) {
			sb.append(iter.next()).append(" ");
		}
		
		output.setValue("msg", "Hello my friends: " + sb.toString());
		
		return SUCCESS;
	}
}
</pre>


Ruby code which is called by the Java action above: /WEB-INF/ruby/users_test.rb 

<pre>

class Users
  
  def get_users
    a = {}
    a['Sergio'] = 1
    a['Robert'] = 2
    a
  end
  
end
</pre>


ApplicationManager:

<pre>
package org.hello;

import org.hello.action.Hello;

public class ApplicationManager extends org.mentawai.core.ApplicationManager {
	
	@Override
	public void loadActions() {
		
		action("/HelloRuby1", Hello.class).on(SUCCESS, "/hello.jsp");
		
		ruby("/HelloRuby2", "MyFirstRubyAction").on(SUCCESS, "/hello.jsp");
		
	}
	
}
</pre>

Source: http://recipes.mentaframework.org/posts/list/57.page
