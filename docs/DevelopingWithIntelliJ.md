Developing with IntelliJ
========================

Many JRuby contributors use [IntelliJ IDEA](https://www.jetbrains.com/idea/) for developing JRuby.

Loading the Project
-------------------

Opening the project is as simple as pointing IntelliJ at our Maven pom.xml.

1. Go to `File => Open`.
2. Browse to your JRuby repository clone.
3. Select `pom.xml` in the root of the clone and proceed.

IntelliJ will open up the project, read in subprojects, and scan all files. You may be asked to set up Maven auto-updating or a Java SDK.

Debugging JRuby
---------------

The easiest way to debug JRuby is to add a run configuration pointing at a known file. Then you can just put the code you want to run in that file and push a button.

1. Go to the `Run` menu (or the dropdown on the related Toolbar with a play button and a bug on it).
2. Select `Edit Configurations`.
3. Add a configuration similar to this (adjust settings and all to your preferences):
   1. Click the `+` to add a configuration.
   2. Add a new "Application" configuration.
   3. Adjust settings.
      * Main class: `org.jruby.Main`
      * VM options: `-Djruby.home=/Users/headius/projects/jruby` plus anything else you want.
      * Program arguments: Only requirement is the file to run or a `-e` line. This is the JRuby command line
      * Working directory: your JRuby clone dir.
      * Use classpath of module: `jruby-core`
      * Before launch: Maven goal `package`

You should be able to Run or Debug the given command line with full stepping, inspection, and so on.