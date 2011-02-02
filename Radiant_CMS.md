Radiant CMS is an elegant, lightweight content management system built upon the popular Ruby on Rails framework and serves eg. the [Ruby Programming Language website](http://www.ruby-lang.org) itself.

For more information on Radiant, please visit the [Radiant website](http://radiantcms.org).

Getting Radiant
---------------

The easiest and recommended way of getting Radiant CMS is via RubyGems.

    jruby -S gem install radiant

Bootstrapping
-------------

In your projects directory issue an application generator command. In this example I'll use the jdbcmysql adapter and my application name will be radiantblog.

    jruby -S radiant --database=mysql radiantblog

To set up the database connections edit radiantblog/config/database.yml and create the database as well.

```yaml
  production:
    adapter: jdbcmysql
    database: radiantblog_production
    username: usr
    password: passwd
    host: localhost
```

You can omit to set up other database configurations for now and correct it later.

    jruby -S rake production db:bootstrap

The bootstrap task will populate your database and set up a basic administration and page system based on your answers to the setup questions in the terminal. To start your application issue the familiar Rails command.

    jruby script/server -e production

Fire up your browser and play around a little with your new CMS - [http://localhost:3000]

Deployment
----------

My first deployment was under a Tomcat 6 with the [[Warbler]] gem.

    jruby -S warble config

My minimal config file is as follows.

```ruby
  Warbler::Config.new do |config|
    config.dirs = %w(cache config db log vendor tmp)
    config.gems += ["activerecord-jdbcmysql-adapter", "radiant"]
    config.gem_dependencies = true
    config.war_name = "ROOT"
  end
```

I set the war name to ROOT to avoid the problems with the default Radiant routing and assets including.

    jruby -S warble:war

Deploy your generated war file under Tomcat's webapps folder, start the server and check http://localhost:8080 for your CMS.
