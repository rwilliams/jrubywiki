### `org.joda.time.DateTime` conflict.
There is a version conflict because both Weblogic and JRuby (1.6.0+) include the JODA time library but the Weblogic version is an older one. To solve it is necessary to tell `config/weblogic.xml` with the following contents

    <weblogic-web-app xmlns="http://www.bea.com/ns/weblogic/weblogic-web-app">
      <container-descriptor>
        <prefer-web-inf-classes>true</prefer-web-inf-classes>
      </container-descriptor>
    </weblogic-web-app>

To include the file in the war include the following line in the [custom configuration](https://github.com/jruby/warbler) file `config/warble.rb`

    config.webinf_files += FileList["config/weblogic.xml"]