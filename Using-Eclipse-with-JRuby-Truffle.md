Eclipse is an alternative IDE for editing JRuby+Truffle. It provides much better feedback from the Truffle DSL annotation processor and actually keeps the project built at all times.

However the build is not shared with Maven to avoid conflicts when Maven builds. Also, as normal JRuby runs from a jar it is not trivial to run from Eclipse compiled .class files.

## Using integrated Maven support

Create a new workspace in Eclipse (>= Luna).

Start by ensuring there is no leftover configuration:
```bash
$ rm -rf {core,truffle}/{.classpath,.settings,.project}
```

We can now import the two projects:
* From the main menu bar, select `File` > `Import...`
* Select `Maven` > `Existing Maven Projects`
* Select `jruby/core` directory as root directory (NOT just jruby)
* Click `Finish`
* Repeat with `jruby/truffle` as root directory

And now enable the Truffle annotation processor:
* Select the `truffle` project
* Right click and choose `Properties`
* Select `Java Compiler` > `Annotation Processing`
* Click on `Enable project specific settings`, then `OK`
* Click `Yes` to rebuild the project

Change the `Output folder` to not interfere with the Maven build for each project:
* Select the `core` project
* Right click and choose `Properties`
* Select `Java Build Path`
* Uncheck `Allow output folders for source folders`
* In `Default output folder:` enter `[PROJECT]/build.eclipse` (with `[PROJECT]` replaced by `core` here)
* Select `Yes` when asked about removing generated resources in the old location
* Repeat with the `truffle` project

You shall be set!

It is possible to run JRuby with a run configuration for `org.jruby.Main.main()`. For that you set the working directory to `${workspace_loc}` in the `Arguments` tab.

Further integration is under investigation.