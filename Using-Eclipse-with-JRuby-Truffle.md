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
* Select `General` > `Existing Projects into Workspace`
* Select `jruby` directory as root directory
* Click `Finish`
* *Unselect* `jruby-jars` and `test`.

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