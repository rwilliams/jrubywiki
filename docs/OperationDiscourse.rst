This page documents efforts to make discourse fully run on JRuby.  The current development branch is: git@github.com:enebo/discourse.git on branch jruby_v2.2.3_spike.

Following basic installation instructions on discourse should nearly work.  Here are a couple of things to note:

 1. mkdir tmp (probably fallout from db:create not working properly)
 1. PASSWORD={my_passord} jruby -S rake db:migrate (skip db:create some issue is making it not work)

I believe there is a manual step of creating the database but I have not run this in a while so I will update this documentation the next time I go through this from scratch process.

To date there are 2 gems which we are working on:

 1. https://github.com/enebo/oj.git (nearly done)
 1. https://github.com/enebo/mini_racer.git

Mini_sql is released but only will work with postgresql.  Making it work with mysql/sqlite3 is future work...

Once that is released I will remove the above git: reference from the Gemfile.

Current status is that it will install and you will be able to log in and see pages but no content is rendering due to a bug in JS (likely something about the port of mini_racer).  Second major issue is mini_racer is built on using isolates between different threads but it currently is one global executor with no locking.