__TOC__

JRuby participated in the JavaOne Script Bowl 2008 with Groovy, Jython, and Scala. JRuby was represented by Charles Oliver Nutter, Groovy was represented by Guillaume Laforge, Jython was represented by Frank Wierzbicki, and Scala was represented by Jorge Ortiz.

==The Challenge==

* #1 - Client Application

Write a simple, read-only Twitter client as a desktop app. The Twitter API is documented at http://groups.google.com/group/twitter-development-talk/web/api-documentation. User provides twitter email address and password , then browses friends and their statuses. User can also filter content based on plain text or a regexp. (Our users are geeks.) Your choice whether to use the JSON or XML output from Twitter.

* #2 - Web Application

As a database, we'll use the "world" sample database from MySQL (http://dev.mysql.com/doc/world-setup/en/world-setup.html). The Web Application lets a user browse countries, sorting them using different criteria (e.g. population or GNP), and select cities and display them on a map using one of the many map widgets around, e.g. Google (http://code.google.com/apis/maps/). For geocoding you could use e.g. http://worldkit.org/geocoder/rest/.

* #3 - Free round

Up to you. Show us something your language can do better than any other! But again, be concise.

==JRuby's Entries==

Charles posted an email on the JRuby list and a blog post on his blog asking for submissions. Because the tasks were quite varied, he wanted to pull in experts from the JRuby community for each category.

===Client Application===

There were three Twitter clients submitted, but one was saved for round three:

# [http://dist.codehaus.org/jruby/talks/Script%20Bowl%202008/twitbucket.zip Twitbucket], by Thomas Enebo of JRuby team and Sun Microsystems, using the [http://ihate.rubyforge.org/profligacy/ Profligacy] GUI framework
# [http://dist.codehaus.org/jruby/talks/Script%20Bowl%202008/twit-and-twitter.zip Twit-and-Twitter], by Logan Barnett, David Koontz, and James Britt of Happy Camper Studios, using the [http://monkeybars.rubyforge.org/ MonkeyBars] GUI framework for JRuby and [http://www.netbeans.org/kb/articles/matisse.html NetBeans Matisse] for the UI layout

JRuby and Groovy were the highest scorers, but JRuby had a slight lead when all votes were counted.

===Web Application===

Only one submission came in for the Rails app:

# [http://dist.codehaus.org/jruby/talks/Script%20Bowl%202008/world.zip "world" application], by Larry Myers of Mapquest

JRuby and Groovy were again the top scorers, with JRuby leading by a similar small margin.

===Free Round===

Two submissions were used for the free round, both based on the [http://github.com/jashkenas/ruby-processing/wikis Ruby-Processing] wrapper around the Processing graphics library.

* [http://dist.codehaus.org/jruby/talks/Script%20Bowl%202008/Nick_Sieger_Twitter_client.zip "space twitter"], by Nick Sieger, a flying-through-space 3D twitter client
* [http://s3.amazonaws.com/jashkenas/ruby_processing_gallery/A%20Face%20for%20Stephen%20Hawking.zip A Face for Stephen Hawking], by Jeremy Ashkenas, an audio-sensitive pulsating flock of color-changing globes

JRuby and Groovy again topped the others, but JRuby's entry garnered twice as many votes as Groovy's discussion of Java annotation support.
