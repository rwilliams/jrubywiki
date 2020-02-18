Getting Started with JRuby
==========================

In this guide, you'll learn how to install JRuby, run a JRuby console, install Gem dependencies, and run some sample code.

Installing JRuby
----------------

For all platforms, make sure you have [Java SE](http://www.oracle.com/technetwork/java/javase/downloads/index.html) installed. You can test this by running the command `java -version`. 

### Microsoft Windows

In your browser, navigate to the [JRuby Downloads page](http://jruby.org/download). Select the "Windows Executable" link for your system (you likely want x64 unless you have a 32-bit Windows installation). Run the installer after it has finished downloading. This will extract the runtime and put the `jruby` command on your `%PATH%` (meaning it will be available from the command-line).

You can confirm that it was installed by opening a command prompt and running:

```
C:\> jruby --version
```

If installed correctly, JRuby will return the current version. You may be prompted with a security warning the first time you run the `jruby` command. This is expected.

### Linux and Mac OS X

On Linux and Mac, you can either install the JRuby binaries or use a Ruby version manager such as [RVM](https://rvm.io/). However, it's best to avoid using a package manager because of issues with keeping the downloaded versions current. If you prefer to build your own JRuby, see [[Downloading JRuby Source and Building It Yourself|DownloadAndBuildJRuby]].

**Note:** On some versions of Linux, you'll need to get the right version of Java installed.

**Note:** If you're on HP-UX, see [[Using JRuby on HPUX|JRubyOnHPUX11_23]].

**Note:** If you're on Alpine Linux (or other Linux that _doesn’t_ use glibc), see [[JRuby on Alpine Linux]].

#### Installing Binaries

In your browser, navigate to the [JRuby Downloads page](http://jruby.org/download). Select the "Binary" package in either `.tar.gz` or `.zip` format. When it's finished downloading, extract it's contents to a `jruby` directory under your home directory. From your home directory, it should look something like this:

```
$ tree -L 1 jruby/
jruby/
├── COPYING
├── LICENSE.RUBY
├── bin
├── lib
├── samples
└── tool

4 directories, 2 files
```

Next put the `bin` directory on your `$PATH`. You can do this temporarily by running the following command:

```
$ export PATH=~/jruby/bin:$PATH
```

Or you can make it permanent by putting that statement in a `.profile` file in your home directory.

Now that JRuby is on your path, you can test that it's working by running:

```
$ jruby --version
```

If set-up properly, JRuby will return the current version.

#### Using RVM

First, make sure you have [RVM installed as outlined in the documentation](https://rvm.io/rvm/install). Then run the following command:

```
$ rvm install jruby
```

If you would like to install a specific version of JRuby, you can replace `jruby` with `jruby-X.Y.Z`, where `X.Y.Z` is a version such as `9.2.9.0` or just `9.2`. 

Once the install process is finished, you can ensure that JRuby is being used like this:

```
$ rvm use jruby
$ jruby --version
```

If it installed correctly, JRuby will return the current version.

Running rake, gem, rails, etc
-----------------------------

The recommended way to run these commands (known as _system-level executable commands_) in JRuby is to **always** use `jruby -S` (it's a standard feature also available with MRI e.g. `ruby -S gem`).

    jruby -S gem list --local
    jruby -S gem install rails activerecord-jdbcmysql-adapter
    jruby -S rails new blog
    cd blog
    jruby -S rake -T
    jruby -S rake db:create db:migrate

The `-S` parameter tells (J)Ruby to use **its** version of the installed binary, as opposed to some other version (such as an MRI version) that might be on your `PATH`.

Running a Ruby Program
----------------------------
To run any other ruby program by using JRuby, run it using the `jruby` command in a command window. For example,

    jruby rails server
    jruby my_ruby_script.rb

**See Also:** [[JRuby Command Line Parameters|JRubyCommandLineParameters]]

jirb: Ruby Interactive Console
-----------------------------
One of the few standard Ruby utilities that has a different name in JRuby than in MRI is the command for the interactive Ruby console: `jirb`. In C Ruby this utility is simply called `irb`. You could also use `jruby -S irb` as described before.

To enable tab completion within *jirb*, add the following line to the configuration file `.irbrc`

    require 'irb/completion'

**Note:** If you're on Linux, BSD, OSX, Solaris, and other UNIXes, the .irbrc file must be in your home directory. If you're on Windows, it goes in your My Documents folder or the folder specified in the HOME environment variable.

**See Also:** [[Jirb Command Line Parameters|JirbCommandLineParameters]]

Installing and Using Ruby Gems
------------------------------

The RubyGems can be easily installed with JRuby with the following command:

    jruby -S gem install rails activerecord-jdbcpostgresql-adapter

Many Gems will work fine in JRuby; however, some Gems build native C libraries as part of their install process. These Gems will not work in JRuby unless the Gem has also provided a Java equivalent to the native library.

Mongrel and [JSON](http://flori.github.io/json/) are two examples of gems that build their native library in a platform independent manner. Each of them specify a parsing library using the Ragel language and a Ragel program can be automatically converted into either C or Java as part of the compile process.

Also, keep in mind that installing gems from behind a firewall will require setting the `HTTP_PROXY` or passing the `http-proxy` argument. For example:

Not authenticated:

    export http_proxy=http://${http-proxy-host}:${http-proxy-port}/
or
    
    jruby -S gem install --http-proxy http://${server}:${port} ${gemname} 

Authenticated:

    export http_proxy=http://{your_user_id}:{your_password}@${http-proxy-host}:${http-proxy-port}/
or

    jruby -S gem install --http-proxy http://${user}:${password}@${server}:${port} ${gemname} 

See also [[JRuby Frequently Asked Questions (FAQs)|FAQs]].

Code Examples
-------------
For some examples of calling JRuby from Java and calling Java from JRuby, see [[JRuby and Java Code Examples|JRubyAndJavaCodeExamples]].
