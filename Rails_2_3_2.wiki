[[Home|&raquo; JRuby Project Wiki Home Page]] &nbsp; &nbsp; [[JRubyOnRails#War_File_Deployment|&raquo; JRuby on Rails: War File Deployment]]
=Rails 2.3.2 Notice=

==English Notice==

Hello all,

Since the Rails 2.3.2 update, session management problems can occur with Ruby using Tomcat or Glassfish when deploying a production Rails application. To resolve the problem, you need to activate your database sessions with JRuby. 

# In the file <tt>initializers/session_store.rb</tt>,  uncomment the line<br/><tt> &nbsp; ActionController::Base.session_store = :active_record_store</tt>.
# Then execute the following Rake commands to deploy the tables that manage sessions:<br/><tt> &nbsp; rake db:sessions:create<br/> &nbsp; rake db:migrate</tt>
# Do a « warble config ».
# Add the following line to tell the Java container that you want to use database sessions:<br/><tt> &nbsp; config.webxml.jruby.session_store = 'db'</tt><br/>This line will modify the file <tt>WEB-INF/web.xml</tt>.
# Finally, in the <tt>environment.rb</tt> file, add the following code above the line <tt>Rails::Initializer.run do |config|</tt>: 
    if defined?(JRUBY_VERSION)
      # hack to fix jruby-rack's incompatibility with rails edge
      module ActionController
        module Session
          class JavaServletStore
            def initialize(app, options={}); end
            def call(env); end
          end
        end
      end
    end

That's all !

==Français==

Bonjour à tous,

Depuis la mise à jour de rails en 2.3.2, lors du déploiement d’une application rails en production avec JRUBY sur Tomcat ou Glassish, certains problèmes se présentent. Pour faire court, c’est la gestion des sessions qui pose problème.
Je vous fais profiter de mon retour car j’ai un peu galérer pour trouver de la doc dessus.

Donc, depuis la version 2.3.2 de rails pour une mise en production avec JRUBY, il faut activer les sessions en base de données.

Dans le fichier <tt>initializers/session_store.rb</tt>, il faut décommenter la ligne
 ActionController::Base.session_store = :active_record_store

Puis, comme sugéré, exécuter les commandes rake suivantes :
 rake db:sessions:create
 rake db:migrate
Ceci, afin de déployer les tables nécessaires à la gestion des sessions.

Après avoir fait un « warble config », il est nécessaire d’ajouter la ligne :
 config.webxml.jruby.session_store = 'db'

Ceci dans le but, d’informer le container java que l’on désire utiliser les sessions en base. Cette ligne aura simplement pour effet de modifier le fichier <tt>WEB-INF/web.xml</tt>.

Puis utiliser un petit hack. Dans le fichier <tt>environment.rb</tt>, au dessus de la ligne <tt>Rails::Initializer.run do |config|</tt>, mettre :
  if defined?(JRUBY_VERSION)
    # hack to fix jruby-rack's incompatibility with rails edge
    module ActionController
      module Session
        class JavaServletStore
          def initialize(app, options={}); end
          def call(env); end
        end
      end
    end
  end

Voilà !
