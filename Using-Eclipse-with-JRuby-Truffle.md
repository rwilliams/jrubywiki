This is an experiment, mainly to have a much better feedback from the Truffle annotation processor.  
(and ideally much faster compile time!)

Maven can be used to generate Eclipse project files:
```bash
mvn eclipse:eclipse
```

Create a new workspace in Eclipse, outside the jruby directory (otherwise Eclipse complains about overlapping dirs).

* From the main menu bar, select File > Import...
* Select General > Existing Projects into Workspace
* Enter the jruby directory in root directory
* Click Finish

Then you need to give the path to your Maven repository (usually ~/.m2/repository).