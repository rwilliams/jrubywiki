= How to use a servlet filter =

You can use servlet filters in your rails-in-a-war-file applications. This page shows you how to use [http://www.ja-sig.org/products/cas/client/javaclient/ Yale's CAS-filter] but you should be able to use other filters as well. 

1. Add the necessary dependencies in config/war.rb

      maven_library 'cas', 'casclient', '2.1.1'
      maven_library 'commons-logging', 'commons-logging', '1.1'

2. Add your filter configuration to the web.xml template in [http://viewvc.rubyforge.mmmultiworks.com/cgi/viewvc.cgi/trunk/rails-integration/plugins/war/lib/create_war.rb?root=jruby-extras&view=markup vendor/plugins/war/lib/create_war.rb] (the web.xml file is regenerated from this template each time you create the war-file). The xml is embedded in the ruby code, search for "def create_webxml".
<pre>
      <filter>
        <filter-name>CAS Filter</filter-name>
        <filter-class>edu.yale.its.tp.cas.client.filter.CASFilter</filter-class>
        <init-param>
          <param-name>edu.yale.its.tp.cas.client.filter.loginUrl</param-name>
          <param-value>https://secure.its.yale.edu/cas/login</param-value>
        </init-param>
        <init-param>
          <param-name>edu.yale.its.tp.cas.client.filter.validateUrl</param-name>
          <param-value>https://secure.its.yale.edu/cas/serviceValidate</param-value>
        </init-param>
        <init-param>
          <param-name>edu.yale.its.tp.cas.client.filter.serverName</param-name>
          <param-value>your server name and port (e.g., www.yale.edu:8080)</param-value>
        </init-param>
      </filter>
      
      <filter-mapping>
        <filter-name>CAS Filter</filter-name>
        <url-pattern>/requires-cas-authetication/*</url-pattern>
      </filter-mapping>
</pre>
3. If you're CAS server use a certificate that has not been signed by a root-ca you need to make the certificate available to your application server. If you use Tomcat you can do this by getting the certificate into a keystore [http://blogs.sun.com/andreas/entry/no_more_unable_to_find] and startup tomcat like this

      startup.bat -Djavax.net.ssl.trustStore=C:/path/to/my/keystore -Djavax.net.ssl.trustStorePassword=changeit
