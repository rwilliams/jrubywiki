It has been tested with Jruby 1.1, rails 2.0.2 and activerecord-jdbc-adapter.

I supose you have jruby installed, then install rails and activerecord-jdbc-adapter

  # gem install rails activerecord-jdbc-adapter

Then copy [http://www.forniol.cat/wp-content/uploads/2008/03/driver-201-full.jar Firebird-jdbc-adapter.jar] into $JRUBY_HOME/lib

Configure your rails application database.yaml like that:

<pre>
development:
  adapter: jdbc
  username: sysdba
  password: masterkey
  driver: org.firebirdsql.jdbc.FBDriver
  url: jdbc:firebirdsql:localhost/3050:database.fdb

</pre>
<b>NOTE</b>: database.fdb is an alias. In order to setup an alias you have to config aliases.conf located on your firebird configuration folder and do something like that:

<pre>  
# /etc/firebird2/aliases.conf

database.fdb = /databases/myblog/myblog.fdb
</pre>

That’s all you need to work with firebird and rails.
