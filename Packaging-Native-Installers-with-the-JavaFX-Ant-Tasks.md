# Introduction

JavaFX brings an awesome packaging tool and terrific set of packaging Ant Tasks to the JVM. Starting with [JDK8](http://jdk8.java.net/download.html), these tools can be used to turn any arbitrary executable jar into native application bundles and installers.  If you can follow the instructions [here](https://github.com/jruby/jruby/wiki/StandaloneJarsAndClasses) for generating a Standalone Executable Jar, then you can turn your JRuby application into an .exe, .msi, .dmg, .deb or .rpm file that can install it and everything it needs to run onto your users systems as a native app. All you need is to get familiar with some ant tasks, and write a Rakefile.

# Prerequesites

[JDK8](http://jdk8.java.net/download.html). Major improvements were made to the JavaFX packaging tools to make them more flexible, flexible enough to work with JRuby jars. These improvements were supposed to go into JDK7 update 10, but didn't make it. So, unless a new update comes out that includes the improved packaging tools, you'll need to download and use the JDK8 Early Access Release.  Efforts are being made to get these tools into [Maven Central](http://search.maven.org/). Once they are there, you can just have rake download the necessary jar (ant-javafx.jar), and not be bothered about versions at all.

**If you are releasing your build script/Rakefile into environments you do not immediately control, your script should check for the JDK 8 and include an appropriate error message. Otherwise you are setting your users up for a great deal of pain.**

The packaging toolkit can only create packages for the OS it is being used on, so for Windows installers you will need to run it on a Windows machine, for OSX installers you will need to run it on a Mac, etc.

In order for the installer to be created, you will need some additional tools installed on your system. For Windows you need either Inno Setup 5 or later for an EXE or [Windows Installer XML (WiX) toolset](http://wix.sourceforge.net/) to generate an MSI. Make sure the WiX toolset's `bin` folder is on the `PATH`.  No special tools are needed to generate a DMG, just a recent version of OSX. For Linux, the packager uses dpkg-deb to create DEB installers and rpmbuild for RPM.

# A Simple Example

There are literally hundreds, perhaps thousands, of tasks and customization options you can choose from, so it would be impossible for me to catalog those all here. Instead, we'll cover a simple example that should explain the basic principles involved, and you can examine the Oracle documentation for more advanced customization.

If you aren't familiar with JRuby's Ant library, [this article](http://blog.engineyard.com/2010/rake-and-ant-together-a-pick-it-n-stick-it-approach) is a good primer, and there is an excellent sub-chapter in the [Using JRuby](http://pragprog.com/book/jruby/using-jruby) book.

```ruby
require 'ant'

task :package_bundle do
  ant do
    taskdef(resource: "com/sun/javafx/tools/ant/antlib.xml",
            uri: "javafx:com.sun.javafx.tools.ant",
            classpath: "${java.home}/../lib/ant-javafx.jar")
    __send__("javafx:com.sun.javafx.tools.ant:deploy", nativeBundles: "all",
             width: "100", height: "100", outdir: "build/",
             outfile: "HelloWorldApp") do
      info(title: "Hello World App", vendor: "Me",
           description: "Test built from Java executable jar")
      application(mainClass: "org.jruby.JarBootstrapMain")
      resources do
        fileset(dir: "dist") do
          include name: "HelloWorldApp.jar"
        end
      end
    end
  end
end
```
Code excerpted from [this answer](http://stackoverflow.com/a/14383886/1200100) by Tom Enebo on http://www.stackoverflow.com .

This is pretty much the most basic Rakefile we could have that would get us a native installer. It's one rake task, containing one big ant block, that then contains several ant tasks within. Let's break it up and take it bit by bit.

`ant do`

The `ant` object will take a block.

```ruby
    taskdef(resource: "com/sun/javafx/tools/ant/antlib.xml",
            uri: "javafx:com.sun.javafx.tools.ant",
            classpath: "${java.home}/../lib/ant-javafx.jar")
```

Because we are using JavaFX's custom ant task's, and not ant's own tasks, we have to tell ant where to find those. This code does that on Ubuntu 12.04 and Windows 7. I have not personally tested it anywhere else.

To explain the next part, I need to show you a little XML.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="HelloWorldApp" default="default" basedir="."
         xmlns:fx="javafx:com.sun.javafx.tools.ant">
//irrelevant xml removed here
<target name="package-bundle">
      <taskdef resource="com/sun/javafx/tools/ant/antlib.xml"
                uri="javafx:com.sun.javafx.tools.ant"
                classpath="${java.home}\..\lib\ant-javafx.jar"/>
      <fx:deploy nativeBundles="all"
                   width="100" height="100"
                   outdir="build/" outfile="HelloWorldApp">
            <info title="Hello World App" vendor="Me"
                     description="Test built from Java executable jar"/>
            <fx:application mainClass="org.jruby.JarBootstrapMain"/>
            <fx:resources>
               <fx:fileset dir="dist">
                  <include name="HelloWorldApp.jar"/>
               </fx:fileset>
            </fx:resources>
      </fx:deploy>
</target>
</project>
```
Notice on the third line where it creates that xml namespace `xmlns:fx`, and how later JavaFX's tasks are all referenced under that namespace, like `fx:deploy` and `fx:application`, this is because the name of many of JavaFX's tasks clash with ant's own built-in tasks. Unfortunately, at this time, JRuby's ant library doesn't yet have a convenient way to accomodate this.

But, ruby is awesome

```ruby
    __send__("javafx:com.sun.javafx.tools.ant:deploy", nativeBundles: "all",
             width: "100", height: "100", outdir: "build/",
             outfile: "HelloWorldApp") do
```

and we can use `__send__` to pass a block to the namespace as an object. One thing I'll mention, if you want to do any customization to your bundle, such as icon, license, etc (and we'll talk a bit about this later), that set of parenthesis after the `__send__` is where you want to put your `verbose: true` .

```ruby
      info(title: "Hello World App", vendor: "Me",
           description: "Test built from Java executable jar")
      application(mainClass: "org.jruby.JarBootstrapMain")
      resources do
        fileset(dir: "dist") do
          include name: "HelloWorldApp.jar"
```
This should all be pretty self explanatory. Info is basic info about your app. mainClass will always be `org.jruby.JarBootstrapMain` for us, unless something in JRuby changes. The resources block is where you list everything that goes into the package and would include additional files like extra jars or other stuff you might need bundled into the installer, although you may already have this all bundled up into your standalone executable jar. Here we just have the one jar, "HelloWorldApp.jar", which is in a directory, `dir`.  There are tons of options you can pass to these tasks, and more tasks you can include. Visit the official [Oracle Ant Tasks Reference](http://docs.oracle.com/javafx/2/deployment/javafx_ant_task_reference.htm) for details.

# Customization

To customize the package, for example to change the icons or license, add `verbose: true` to the `deploy` task.

```ruby
    __send__("javafx:com.sun.javafx.tools.ant:deploy", nativeBundles: "all",
             width: "100", height: "100", outdir: "build/",
             outfile: "HelloWorldApp", verbose: true) do
```

This will cause the JavaFX packaging tools to enter verbose mode, and provide more details about the process, including (the important part for customization) the location of a temporary folder where the config resources for the build are held and a list of the resources and the role of each. Copy the contents of this tmp folder into a folder in your project directory (the dir you run rake from) where the packaging tools will know to look for them. For example, on linux this would be `main_project_dir/package/linux`. On OSX, it is `main_project_dir/package/macosx`. So, if I wanted to use a custom icon, I'd replace the default icon with my own, ensuring it has the same name, and place it inside that linux or macosx folder.  Then run the build again. You can find more information on customizing at the [official Oracle documentation](http://docs.oracle.com/javafx/2/deployment/self-contained-packaging.htm#BCGICFDB).  [This blog post](http://ed4becky.net/homepage/javafx-from-the-trenches-part-1-native-packaging/4/) may also be helpful, as he goes through the process of customizing an app for both Windows and OSX.

# For Addition Resources See:

[Oracle JavaFX Docs: Self-Contained Application Packaging](http://docs.oracle.com/javafx/2/deployment/self-contained-packaging.htm)

For an example of using these tasks programatically, check out the [JRubyFX](https://github.com/nahi/jrubyfx) project's jrubyfx-jarify tool, which takes a few command-line options and spits out an executable jar and a native package.