Here is a list of sites and projects using JRuby. Please add descriptions if possible. 

Use this structure as a guide (but feel free to omit or add information as appropriate).

## NAME OF PROJECT
Description of project.

### Why JRuby?
Explain why JRuby is being used.

### How is JRuby Used?
Explain how JRuby is being used.

### Java Integration
Describe how the project uses JRuby's Java integration, if relevant.

### Regrets and Recommendations
List any regrets choosing JRuby or recommendations for others (and accentuate the positive).

### Project Metrics
Information on hits/day, transactions/s, and other metrics that would help others evaluate JRuby for their projects.



## Fleet control at Oslo Airport Gardermoen
The project monitors and controls the passenger busses and fuel trucks at Oslo Airport Gardermoen.  Each vehicle contains an embedded device with GPS and WIFI that reports the activities of the vehicle to a central server.  Flight list and orders are received by integrating the server to the airport flight database and a central office dispatch unit sends assignments to the vehicles.  All assignments are stored on the server and are reported to the airport authorities through the office backend system.  A web application for analysis and reporting offers detailed drill down and charts.

### Why JRuby?
We had an extremely tight schedule and needed a language that let us write the features quickly.  Ruby fit that need.  At the same time we absolutely needed a tried production platform that would scale well.  Java fit that need.  JRuby seemed a tempting alternative, and all experiments, proof of concept, pre-project and initial iterations of the project proved JRuby to be a really great platform for developing complex and critical applications.

A REALLY important factor that was tested early in the project was the responsiveness of the JRuby team.  The support from the JRuby team has been absolutely fantastic.  Since project start in 2009 and deployment to production in 2010, there have been bugs of differing severities, and each time the JRuby team has responded quickly and reliably.  Over time we have become involved in the JRuby project and we are confident that any problems that occur will be solved.

### How is JRuby Used?
JRuby is used in every tier of the system and large parts of the code is shared across the embedded systems (Android and Linux), thick client office systems (Eclipse RCP), the web apps (Rails), and the communication server (headless JRuby process).

### Java Integration
In the fuel trucks, we use JNI for reading GPIO.  All tiers use ActiveRecord JDBC Adapter for database access, DerbyDB in the client systems, and PostgreSQL on the server.  ActiveMQ is used for integrating to an automated airline ticket reporting system.  The client systems use Eclipse SWT, JFace, and RCP.  Apache POI is used to import Office documents.  Lots more Java libraries are used and work well.  Being able to select from the Java ecosystem in addition to the Ruby gems has proven very valuable.

### Regrets and Recommendations
We have no regrets in choosing JRuby.  It is and has been a very pleasant and productive platform to work on.  As with all projects, be sure to attack your integration points and high-risk parts of your system first.  Test performance.

### Project Metrics
We have around 200 users, 600 assignments per day, 50 vehicles.  The web apps seldom have more that 10 hits/s.



## [OpenTelegard/2](http://telegard.org)
OpenSource implementation of the Telegard BBS.

### Why JRuby?
JRuby offered advantages of many existing Java libraries not available for MRI Ruby.

### How is JRuby being used?
JRuby is the primary language for the project.

### Other
Project runs on OpenJDK 6

## [UploadBooth.com](http://uploadbooth.com/about UploadBooth.com)
UploadBooth is a service that enables you to upload and share tagged files with others online in a simple way. You can start using UploadBooth right away by uploading your first batch of files.

## [blissmessage.com](http://www.blissmessage.com)
BlissMessage is an iPhone app which lets you send free text messages from iPhone to iPhone and from the web to iPhones as well. It leverages push notifications for instant delivery of the messages. While the basic use case is sending free text messages to other users, you can also get notified about your Twitter replies and mentions if you enter your Twitter name in the profile.

### Why JRuby?
Easy and performant integration of mature MQ technologies (ActiveMQ). Java is a high-performant language to fall back on in case, and it's safer for me to code Java than C.

### How is JRuby being used?
NGINX - Jetty - JRuby-Rack

## [PONS.eu](http://pons.eu)
Multi-language high-performance online dictionary designed to serve millions of search requests a day. The system combines high-quality content by the renowned dictionary publisher PONS with content generated by the web's crowd intelligence. The scalable system is developed and hosted by [finnlabs](http://finn.de/2009/5/1/online-dictionary-pons-eu).

### Why JRuby?
To share large in-process caches and lucene indices between multiple rails instances and to reuse Java libraries.

### How is JRuby being used?
Optimized Java App handles perfomance critical dictionary search requests, whilst JRuby on Rails runs everything else.

### Java integration?
Yes

### Project metrics
Millions of search requests a day.

## [SpringBook](http://labs.openmaru.com/projects/springbook/pages/484920?lang=en)

## [OpenWFEru](http://openwferu.rubyforge.org)

## [Project Kenai](http://kenai.com)
Open Source, Student and Developer Collaboration Site for Software Projects

### How is JRuby being used?
JRuby on Rails running on Glassfish

### Project metrics
~15K/day (as of 2008-10-29) and growing

## [Mingle](http://studios.thoughtworks.com/mingle-project-intelligence)

[Oracle Mix](http://mix.oracle.com)
Running on Oracle Application Server, Oracle Database, Oracle Internet Directory,
Oracle SSO, and JRuby 1.1RC2 on Rails (1.2.6). Built by the Oracle AppsLab in
collaboration with ThoughtWorks.

### Java integration?
Yes


## [JRubyArt](https://ruby-processing.github.io/JRubyArt/)
[JRubyArt](https://ruby-processing.github.io/JRubyArt/) is a wrapper for Processing-3.0, a visual coding sketchbook (includes audio, 3D graphics, export to 3D printers etc...).

### Why jruby?
It just works, and the performance hit (vs vanilla processing) isn't too bad!

### How is JRuby being used?
To access the Processing core and other java libraries, and to integrate ruby gems into processing sketches.

### Java integration?
Yes, projects include custom jruby extensions (compiled using polyglot maven). But also includes the dynamic loading of native binaries (required at runtime) using a custom library loader when needed.


## [Desktop Application Development in JRuby](http://spin.atomicobject.com/2007/11/12/desktop-application-development-in-jruby)

## [Sonar](http://sonar.hortis.ch)
An open-source entreprise dashboard which gauges quality of Java applications through
the observance of coding rules conventions, metric measures and advanced indicators.

### Why JRuby?
Embed Ruby/Rails into a JEE application

### How is JRuby being used?
Jetty server + goldspike + activerecord-jdbc-adapter. Build with maven.

### Java integration?
Yes

## [Mediacast](http://mediacast.sun.com)
A (media)file hosting application. [Development and deployment details](http://blog.igorminar.com/2008/01/jruby-on-rails-rewrite-of.html)

### Java integration?
Yes

## [Medienservice Sachsen](http://www.medienservice.sachsen.de/)
The local government of Saxony (Germany) publishes all its press releases via this application.

### Why JRuby?
App built on Rails, target environment is J2EE-only.

### How is JRuby being used?
Running Rails, acessing Java libraries from the Rails app.

### Java integration?
Yes

## [schnell](http://code.google.com/p/schnell-jruby/)
schnell is a zippy (Fast), yippy (Easy) and hippy (Cool) tool for testing websites.

### Why JRuby?
Written on jruby using webdriver (formerly htmlunit).

### How is JRuby being used?
schnell is a headless browser web application testing tool and uses webdriver (formerly htmlunit) as its driver.

### Java integration?
Yes


## [TriSano](http://www.trisano.org TriSano)
TriSano&acirc;&bdquo;&cent; is an open source, citizen-focused surveillance and
outbreak management system for infectious disease, environmental hazards, and
bioterrorism attacks. It allows local, state and federal entities to track, control and
ultimately prevent illness and death.

### Why JRuby?
Extensive roadmap - didn't think Ruby alone could get it done and want to fall back
to Java rather than C.

### How is JRuby being used?
Running Rails

### Java integration?
Not yet, but will likely soon

## [Auktionskompaniet.com](http://www.auktionskompaniet.com)
Auktionskompaniet.com is a Swedish chain of auction houses owned by Bukowskis
Auktioner AB, Swedens biggest auction house. Auktionskompaniet hold auctions with objects
in the average consumer price range.

### Why JRuby?
Third-party library integration

### How is JRuby being used?
Rails (2.1.2), Warbler 0.9.11, Glassfish v2, Ubuntu 8.04

### Java integration?
Yes

### Project metrics
~100k page views / day (2008-10)

## [SCRIP-SAFE](http://scrip-safe.com)
Secure transcript delivery system developed by [Edge Case](http://theedgecase.com).

### Why JRuby?
to make use of java libraries

### How is JRuby being used?
Rails

## [HomeNet Profiler](http://cmon.lip6.fr/hnp/)
Measurements in home networks, for research purposes.

### Why JRuby?
Portability with a GUI. Java libraries.

### How is JRuby being used?
Executable JAR

### Java integration?
Yes

### Project metrics
Successfuly ran 2500 runs on more than 2100 heterogeneous machines (OSes/countries etc.).


## [eazyBI](https://eazybi.com/)
eazyBI is easy to use business intelligence web application for analyzing data either from uploaded files or from other source applications.

### Why JRuby?
1. to integrate [Mondrian OLAP](http://mondrian.pentaho.com/) Java library using [mondrian-olap](https://github.com/rsim/mondrian-olap) gem
2. to create compiled standalone war package for on-site application deployment

### How is JRuby being used
Primary language to develop Ruby on Rails 3.x application. Production deployment using Kirk or remote deployment using embedded Winstone or using Tomcat or other Java app servers.

### Java integration
OpenJDK 6

### Regrets and recommendations
Almost none (just slower development and test startup time compared with MRI)

## [The Currency Cloud](http://www.thecurrencycloud.com/)
The Currency Cloud delivers Cross Border Payments as a Service.

## [Jux](https://jux.com)
Jux aims to be the best showcase for your content.  You can post photos, videos, articles, slideshows, etc.

### Why JRuby?
We wanted to move to JRuby from MRI Ruby so we could rewrite some of our slow, CPU-bound code in Clojure.

### How is JRuby Used?
We run Rails on JRuby.

### Java Integration
Although not Java, we are using JRuby's Java integration to call from JRuby into Clojure.

### Regrets and Recommendations

Downsides:

Rails asset recompilation during development is slower.

Cannot use newrelic_rpm gem as it does not clean up thread locals which causes excessive memory usage with tomcat.  This is not a problem in TorqueBox, however.  I suspect JBoss clears out thread locals after each request.

~~Single sign-on with desk.com has stopped working. We have traced this down to what we believe to be an SSL bug in jruby.  See http://jira.codehaus.org/browse/JRUBY-6951~~  Fixed in 80ba2ff470d4748a1836d40afac0b879b9a0d943 !

Recommendations:

Do simulate load on a test setup before deploying to production.  We didn't do so at first and we suffered severe performance degradation due to file descriptor and memory leaks.

We recommend Eclipse Memory Analyzer tool for debugging memory leaks.  Gems not cleaning up thread locals has been the biggest source of memory leaks for us.

Avoid haml.  Besides being slower than erb and slim and not supporting streaming, haml uses exceptions for control flow (fixed in upcoming 3.2.0) and changes the values of constants, both of which cause performance issues in jruby.

Be careful with threadsafe mode, especially if you are using a large number of third-party gems.  Many gem authors do not code their gems with thread safety in mind.  JRuby is raising awareness, however.

Mongoid 3 will leak file descriptors on tomcat-based deployments.  This is because [Mongoid 3 stores sessions as a thread local](https://github.com/mongoid/mongoid/issues/2369).  You can work around this by running in non-threadsafe mode and patching Mongoid to [store sessions as a global variable](https://github.com/tolsen/mongoid/commit/cec363b2bd74c1b6fed0ce4419a3cecc50e9ac94). 

### Project Metrics
1.4M hits / day.  105K users.

## Nottingham Infomation Prescriptions

An Information Prescription is a special kind of prescription which provides information, rather than tablets or medicines. The website is a service run by the National Health Service, UK.

### Why JRuby?

Ability to use Java libraries such as flying Saucer/iText and easily integrate with JBOSS via the Torquebox project. So we can turn threading on in Rails.

### How is JRuby Used?

To run a RubyOnRails 3.0 app inside JBoss.
PDF generation, XML cleaning

### Regrets and Recommendations

Starting up the JVM each time for running test can be slow, especially when its sub-second on MRI, but there is shotgun which may solve this problem.

I'd recommend not starting with Torquebox from day 1 and only adding it when needed. There is a Torquebox Lite (which is web only) or just run with Puma (we do this for some secondary-websites). Once you need JBoss specific features add Torquebox.

### Project Metrics
~150,000 visitors per month / 140 users (admins, healthcare professionals, editors)


## [NikaMail](https://nika.run)
Portable, extendable, zero-config email server for JVM and Ruby. *NikaMail* is a SMTP and POP3 server with management interface and e-mail automation capabilities.

### Why JRuby?
Software written in JRuby is portable. 

### How is JRuby Used?
1. NikaMail debug console is actual IRB. 
2. Web interface is written with Sinatra. 
3. MTA is written in pure Ruby. 
4. Email post-processing, parsing, relaying.

### Java Integration
NikaMail is built on top of [Mireka](http://mireka.org/) and [SubEtha](https://github.com/voodoodyne/subetha). Some extensions are written in Java. More than 85% of NikaMail code interacts with Java libraries.

### Regrets and Recommendations
Developers w/o java experience need time to understand some patterns.

### Project Metrics
~1500 Excel and PDF reports per day, ~6000 outbound emails and notifications per day. 
