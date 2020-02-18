this is for the **jruby-openssl-0.9.5.gem** (and newer). the gem allows Bouncy-Castle version **1.47**,**1.48**,**1.49**,**1.50** and comes with **1.47** embedded inside the gem. in case you want to use another version of the Bouncy Castle jars then **JBundler** offers a way to do so. create a **Jarfile** like this

    BC_VERSION = '1.50'
    jar 'org.bouncycastle:bcpkix-jdk15on', BC_VERSION
    jar 'org.bouncycastle:bcprov-jdk15on', BC_VERSION

add to your **Gemfile**

    gem 'jbundler', '~> 0.6'

update your installed gems + jars

    $ jbundle install

and now you can verify that you are using

    $ bundle exec ruby -rjbundler -e "p Java::OrgBouncycastleJceProvider::BouncyCastleProvider.new.info"

or with your application. just make sure you use ```require 'jbundler'``` as early as possible (bundler's auto require feature should(?) take care of this)