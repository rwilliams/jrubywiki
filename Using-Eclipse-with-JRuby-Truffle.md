This is an experiment, mainly to have a much better feedback from the Truffle annotation processor and ideally much faster compile time!

## Using integrated Maven support

Create a new workspace in Eclipse (>= Luna).

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

You might want to change the `Output folder` in `Java Build Path` so it is better separated from the Maven compilation process.

It is possible to run JRuby with a run configuration for `org.jruby.Main.main()`.  
Further integration is under investigation.