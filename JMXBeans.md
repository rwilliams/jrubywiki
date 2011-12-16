JRuby can take advantage of the wonderful facilities that are part of the Java environment. One such facility is the use of JMXBeans for viewing the Ruby stack traces of a running program. To achieve this, there are a few prerequisites.

1. JDK7 for Java7 (contains visualvm)
2. JRuby 1.7.0 or github master


To get access to the Ruby stack traces, let's first setup visualvm. Launch visualvm from the command line.
```
    % jvisualvm
```
Once inside visualvm, go to the Tools menu and select the Plugins item. In the window that pops up, there should be a list of Beans that can be enabled in the Available Plugins tab. Select 'VisualVM-MBeans' and click on the Install button.

After installation, start up your JRuby application with the --manage switch to enable the MBean in the runtime.
```
    % jruby --manage script.rb
```

In VisualVM, double-click on the JRuby runtime listed in the left-most pane. When the VM is opened, there should be an additional tab on the far right called MBeans. Select that tab.

The tab contains a list of elements that can be expanded. Expand the org.jruby -> Runtime -> "PID" (this will be an integer). Inside the final element, select Runtime. In the right side of the pane should be several buttons that will produce Ruby thread dumps for the running program.