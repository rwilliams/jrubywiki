Getting Started with JRuby
==========================

To get JRuby on your system, install the most recent JRuby binary file for your system, extract it, and then work with it from the command line in a terminal window or command window

Installing JRuby
----------------
The easiest way to get up and running with JRuby is to download the latest binary, extract it, and add the directory to your PATH environment variable.

**Note:** It's best to avoid using a package manager because of issues with keeping the downloaded versions current.

If you prefer to build your own JRuby, see [[Downloading JRuby Source and Building It Yourself|DownloadAndBuildJRuby]].

To download and install JRuby: [Download a JRuby binary file](http://jruby.org/download) 

* For OSX, Linux, BSD, Solaris, and other UNIX varieties, get the most recent `jruby-bin-X.Y.Z.tar.gz` file.
* If you're on Microsoft Windows, get the most recent `jruby-bin-X.Y.Z.zip` file.
* Extract JRuby into a directory.
* Add that directory's bin subdirectory to the ''end'' of your `PATH` environment variable.
* On OSX, Linux, BSD, Solaris, and other UNIXes, the variable is `$PATH`, and a sample JRuby path is `/opt/jruby/bin`.
* On Microsoft Windows, the variable is `%PATH%`, and a sample JRuby path is `C:\JRuby\jruby-1.5.0\bin`. Also, make sure your `JAVA_HOME` environment variable points to your Java installation. For example, `C:\Program Files\Java\jdk1.6.0_14\`.

**Note:** On some versions of Linux, you'll need to get the right version of Java installed. For more infomation, see [[JRuby With Wrong Java|JRubyOnUbuntu]].

**Note:** If you're on HP-UX, see [[Using JRuby on HPUX|JRubyOnHPUX11_23]].

### Linux and OSX Installation Example
Once you've downloaded or built a JRuby installation and it is located in the directory `/opt/jruby`, you'll need to add `/opt/jruby/bin` to the end of your `$PATH` environment variable.

On Mac OS X and Linux, you can add to the `PATH` variable with the export command

    export PATH=$PATH:/opt/jruby/bin

### Microsoft Windows XP Installation Example
Once you've downloaded or built a JRuby installation and it is located in the directory `C:\JRuby\jruby1.5.0\`, you'll need to add `C:\JRuby\jruby1.5.0\bin` to the end of your `%PATH%` environment variable. You'll also need to ensure that your `JAVA_HOME` variable is set to the location of your current Java installation, for example, `C:\Program Files\Java\jdk1.6.0_14\`.

* In Windows XP, choose Start > Control Panel > System to open the System Properties window.
* Click the Advanced tab, then click the Environment Variables button at the bottom of the window.
* In the System Variables section, scroll down to `Path`, select it, and click Edit.
* Add the path to the JRuby bin directory to the end of the Path. For example, add:<br/>;`C:\JRuby\jruby1.5.0\bin`
* Look for `JAVA_HOME` in both the User Variables section and the System Variables section and make sure it points to your current Java installation. If necessary, create a new JAVA_HOME variable as follows:
* Under the User Variables section, click New.
* **Variable name:** enter `JAVA_HOME`
* **Variable value:** enter the actual path. For example, <br/> `C:\Program Files\Java\jdk1.6.0_14\`.
* Click OK to add the variable.
* Click OK at the bottom of the Environment Variables window.
* Click OK to close the System Properties window.

Did It Work?
-----------
To test whether JRuby installed correctly, open a command window or terminal window and run:

    jruby -v

If it installed correctly, JRuby will return the current version.

How Do I Run rake, gem, etc?
----------------------------
The recommended way to run these commands (known as _system-level executable commands_) in JRuby is to **always** use `jruby -S`.

    jruby -S gem list --local
    jruby -S gem install rails mongrel jdbc-mysql activerecord-jdbcmysql-adapter
    jruby -S rails blog
    cd blog
    jruby -S rake -T
    jruby -S rake db:migrate

The `-S` parameter tells JRuby to use **its** version of the installed binary.

How Do I Run a Ruby Program?
----------------------------
To run any other ruby program by using JRuby, run it using the `jruby` command in a command window. For example,

    jruby script/server
    jruby my_ruby_script.rb

**See Also:** [[JRubyCommandLineParameters|JRuby Command Line Parameters]]

jirb: Ruby Interactive Console
-----------------------------
One of the few standard Ruby utilities that has a different name in JRuby than in C Ruby is the command for the interactive Ruby console: `**jirb**`. In C Ruby this utility is simply called `irb`.

To enable tab completion within jirb, add the following line to the configuration file .irbrc:
  require 'irb/completion'

**Note:** If you're on Linux, BSD, OSX, Solaris, and other UNIXes, the .irbrc file must be in your home directory. If you're on Windows, it goes in your My Documents folder or the folder specified in the HOME environment variable.

**See Also:** [[JirbCommandLineParameters|Jirb Command Line Parameters]]

Installing and Using Ruby Gems
------------------------------
The RubyGems can be easily installed with JRuby with the following command:

    jruby -S gem install rails mongrel jdbc-mysql activerecord-jdbcmysql-adapter

Many Gems will work fine in JRuby; however, some Gems build native C libraries as part of their install process. These Gems will not work in JRuby unless the Gem has also provided a Java equivalent to the native library. 

Mongrel and Hpricot are two examples of Gems that build their native library in a platform independent manner. Each of them specify a parsing library using the Ragel language and a Ragel program can be automatically converted into either C or Java as part of the compile process.

Also, keep in mind that installing gems from behind a firewall will require setting the `HTTP_PROXY`. For example:

Not authenticated:

    export http_proxy=http://${http-proxy-host}:${http-proxy-port}/

Authenticated:

    export http_proxy=http://{your_user_id}:{your_password}@${http-proxy-host}:${http-proxy-port}/

See also [[FAQs|JRuby Frequently Asked Questions (FAQs)]].

Code Examples
-------------
For some examples of calling JRuby from Java and calling Java from JRuby, see [[JRubyAndJavaCodeExamples|JRuby and Java Code Examples]].
