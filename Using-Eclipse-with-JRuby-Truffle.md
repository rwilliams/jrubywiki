Eclipse is an alternative IDE for editing JRuby+Truffle. It provides much better feedback from the Truffle DSL annotation processor and actually keeps the project built at all times.

First, make sure the project is already built from the command line:
```bash
$ mvn
```

### Using integrated Maven support

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

And now enable the Truffle annotation processor.  
This depends on your branch:  
_**on master**_
* Select the `truffle` project
* Right click and choose `Properties`
* Select `Java Compiler` > `Annotation Processing`
* Click on `Enable project specific settings`,
* Add `lib/jruby-truffle.jar` using `Add External JARs`, on the submenu `Factory Path`
* Click `Ok` and `Yes` to rebuild the project

_**on truffle-head**_
* Install the [m2e-apt](https://marketplace.eclipse.org/content/m2e-apt) plugin.
* Select the `truffle` project
* Right click and choose `Properties`
* Select `Maven` > `Annotation Processing`
* Click on `Enable project specific settings`,
* Select the first option: "Automatically configure the JDT APT"
* Click `Ok` and `Yes` to rebuild the project

We must now import the layout generated files.  
* Select the `truffle` project
* Expand in the file hierarchy: `truffle` then inside `target` and `generated-sources`.
* Right click on the `generated-sources` folder and select `Build Path` > `Use as Source Folder`  
Then,
* Select the `truffle` project
* Right click and choose `Properties`
* Select `Java Build Path` and the `Source` tab.
* In the list find `truffle/target/generated-sources`
* Select two lines below `Included: (All)`
* Click `Edit` on the right side.
* Click the `Add` button on the right side of `Inclusion patterns`
* Click `Browse...`
* Expand the file hierarchy: `org` then `jruby`, then `truffle` and `runtime`
* Select the `runtime` folder and click `OK`, `OK` again and `Finish` and a final `OK`.

Change the `Output folder` to not interfere with the Maven build for each project:
* Select the `core` project
* Right click and choose `Properties`
* Select `Java Build Path` and the `Source` tab.
* Uncheck `Allow output folders for source folders`
* In `Default output folder:` enter `[PROJECT]/build.eclipse` (with `[PROJECT]` replaced by `core` here)
* Select `Yes` when asked about removing generated resources in the old location
* Repeat with the `truffle` project

You shall be set!

### Running from the Eclipse files directly

Use the `tool/jruby_eclipse` script to run directly from Eclipse .class files.  
Any change in truffle Java or Ruby files will be available as soon as Eclipse finishes compiling.

```bash
$ tool/jruby_eclipse -X+T -e 'p RUBY_ENGINE'
"jruby+truffle"
```

The [jt workflow tool](https://github.com/jruby/jruby/tree/master/truffle#workflow-tool)
automatically picks up that script over `bin/jruby` if you have  
`export JRUBY_ECLIPSE=true` in your environment.

### Ruby support
You can use Aptana Rails to be able to edit ruby files inside Eclipse. See http://stackoverflow.com/questions/12366924/how-to-use-eclipse-for-ruby-on-rails-ror