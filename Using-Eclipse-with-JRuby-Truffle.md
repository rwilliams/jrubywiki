This is an experiment, mainly to have a much better feedback from the Truffle annotation processor and ideally much faster compile time!

## Using integrated Maven support

Create a new workspace in Eclipse.

Start by ensuring there is no leftover configuration:
```bash
$ rm -rf core/.classpath core/.settings core/.project
```

We can now import the project:
* From the main menu bar, select `File` > `Import...`
* Select `Maven` > `Existing Maven Projects`
* Select `jruby/core` directory as root directory (NOT just jruby)
* Click `Finish`

And now enable the Truffle annotation processor:
* From the main menu bar, select `Project` > `Properties`
* Select `Java Compiler` > `Annotation Processing`
* Click on `Enable project specific settings`, then `OK`
* Click `Yes` to rebuild the project.

You shall be set!

It is possible to run JRuby with a run configuration for Main.main().
Further integration is under investigation.

#### Old technique not using Maven support

Be sure to have the project built before with `mvn`.

Maven can be used to generate Eclipse project files:
```bash
mvn eclipse:eclipse
```

Create a new workspace in Eclipse, outside the jruby directory (otherwise Eclipse complains about overlapping dirs).

We need to set the path to your Maven repository (usually `~/.m2/repository`):
* From the menu, select Window > Preferences
* In the search field type `Classpath Variables`
* Click on `New` and enter the variable name: `M2_REPO`
* Click on `Folder...` and select your home, then `.m2` then `repository`

We can now import the project:
* From the main menu bar, select `File` > `Import...`
* Select `General` > `Existing Projects into Workspace`
* Enter the jruby directory in root directory
* Click `Finish`

Close the `jruby-tests` project since the setup is wrong for now.

There are some for now unsolved errors in JRuby generated files, so in the `Package Explorer`,
open `target/generated-sources`, then right click on `org.jruby` and select `Build Path` > `Exclude`.

Now it should start to work to some extent!
