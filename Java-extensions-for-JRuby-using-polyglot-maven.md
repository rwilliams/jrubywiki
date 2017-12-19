Maven has become the _de-facto_ standard for managing jar dependencies on java
projects and thanks to [polyglot maven][] it can now do the same for jruby projects 
with java extensions. Over at [jruby-examples][] we have created a simple project that 
outlines the necessary steps.
You can get the latest release of maven (apache-maven-3.5.2) [here][], you need at least 
version-3.3.1 for polyglot maven.

In your project directory create a `.mvn` folder and create `extensions.xml`
in that folder (this lets maven know that it should expect a `pom.rb` file, for
polyglot-ruby rather than a `pom.xml` for regular maven). Before you do this you should probably clone and build our example project (navigate to extensions/basic/jruby-ext and `rake`  to compile and test).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<extensions>
  <extension>
    <groupId>io.takari.polyglot</groupId>
    <artifactId>polyglot-ruby</artifactId>
    <version>0.1.19</version>
  </extension>
</extensions>
```

Here is the complete `pom.rb` which as you see is just ruby (we generate the `pom.xml` for completeness but it does not get used in the build)

```ruby
project 'jruby-ext' do

  model_version '4.0.0'
  id 'com.purbon:jruby-ext:1.0'
  packaging 'jar'
  
  description 'example JRuby extension'
  
  developer 'purbon' do
    name 'Pere Urbón'
    email 'pere.urbon@gmail.com'
    roles 'developer'
  end
  
  issue_management 'https://github.com/jruby/jruby-examples/issues', 'Github'
  
  source_control(
    url: 'https://github.com/jruby/jruby-examples',
    connection: 'scm:git:git://github.com/jruby/jruby-examples.git',
    developer_connection: 'git@github.com:jruby/jruby-examples.git'
  )
  
  properties(
    'maven.compiler.source' => '1.7',
    'maven.compiler.target' => '1.7',
    'source.directory' => 'src/main/java', # poxy Eclipse folders
    'project.build.sourceEncoding' => 'utf-8',
    'polyglot.dump.pom' => 'pom.xml',
    'jruby.api' => 'http://jruby.org/apidocs/',
  )

  jar 'org.jruby:jruby:9.1.5.0'
  
  plugin_management do
    plugin :resources, '2.6'
    plugin :dependency, '2.8'
    plugin(
      :compiler, '3.5.1',
      source: '${maven.compiler.source}',
      target: '${maven.compiler.target}'
    )
    plugin(
      :javadoc, '2.10.4',
      detect_offline_links: 'false',
      links: ['${jruby.api}']
    )
    plugin(
      :jar, '3.0.2',      
      archive: {
        manifestFile: 'MANIFEST.MF' # camel case reqd
      }    
    )
  end
  
  build do
    default_goal 'package'
    source_directory '${source.directory}'
    final_name 'jruby-ext'
  end
end
```

This build contain a couple of advanced features (including a `MANIFEST.MF` file that we generate in our Rakefile, and javadoc, including the jruby-api, without which it would be useless).

To generate the javadoc:

```bash
mvn javadoc:javadoc
```

The beauty of the maven build system is that it readily manages other jar dependencies, especially if they are available from maven central. See for example [ruby-processing][] that depends on processing-2.2.1 `core` and `video` jars as well as jruby. Until we fill out our [documentation][] you may be interested in these two guides by [James Coglan][] and [Richard Huang][] to building jruby-extensions (further they are also real world examples using `BasicLibraryService`, smart arses like the JRuby team use a slightly different approach, but wish to promote a single method for general use).

[polyglot maven]:https://github.com/takari/polyglot-maven
[here]:https://maven.apache.org/download.cgi
[jruby-examples]:https://github.com/jruby/jruby-examples
[ruby-processing]:https://github.com/jashkenas/ruby-processing/blob/master/pom.rb
[Richard Huang]:http://blog.huangzhimin.com/2012/08/23/how-to-write-a-jruby-gem-part-2/
[James Coglan]:https://blog.jcoglan.com/2012/08/02/your-first-ruby-native-extension-java/
[documentation]:https://github.com/jruby/jruby/wiki/Method-Signatures-and-Annotations-in-JRuby-extensions