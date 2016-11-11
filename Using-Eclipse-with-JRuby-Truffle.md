Eclipse is an alternative IDE for editing JRuby+Truffle. It provides much better feedback from the Truffle DSL annotation processor and actually keeps the project built at all times.

First, make sure the project is already built from the command line:
```bash
$ mx build
```

### Generate the project files

```bash
$ mx eclipseinit
```

### Import the projects

Create a new workspace in Eclipse (>= Luna).

We can now import the projects:
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
* Click on `Enable project specific settings`
* Click `OK` and `Yes` to rebuild the project

Follow the *common steps* below.

_**on truffle-head**_
* Install the [m2e-apt](https://marketplace.eclipse.org/content/m2e-apt) plugin (drag and drop the `Install` button).
* Select the `truffle` project
* Right click and choose `Properties`
* Select `Maven` > `Annotation Processing`
* Click on `Enable project specific settings`,
* Select the first option: "Automatically configure the JDT APT"
* Click `Ok` and `Yes` to rebuild the project

_**common steps**_

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