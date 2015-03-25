---
layout: post
title: "Проблемы с апдейтом bundler 1.9"
date: 2015-03-25 11:56:42 +0200
comments: true
author: Sergey Gernyak
categories: [rvm, bundler, 1.9, server crash, rvm-capistrano]
---

Несколько дней назад вышел апдейт известного гема для менеджмента
зависимостей __bundler__. Вот [пруф линк](http://bundler.io/blog/2015/03/21/hello-bundler-19.html).

Я сразу же поспешил обновить его, т.к. у меня, как оказалось, стояла еще
версия 1.7.7. Но оказалось не все так радостно.

После апдейта перестали запускаться rails приложения. Вывод после выполнения команды `rails s` стал следующим:

```
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/values/time_zone.rb:283: warning: circular argument reference - now
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/newrelic_rpm-3.8.1.221/lib/new_relic/agent/configuration/default_source.rb:674: warning: duplicated key at line 985 ignored: :disable_mongo
bin/rails:6: warning: already initialized constant APP_PATH
/Users/sergio/Work/railsapps/tiporbet/bin/rails:6: warning: previous definition of APP_PATH was here
Usage: rails COMMAND [ARGS]

The most common rails commands are:
 generate    Generate new code (short-cut alias: "g")
 console     Start the Rails console (short-cut alias: "c")
 server      Start the Rails server (short-cut alias: "s")
 dbconsole   Start a console for the database specified in config/database.yml
             (short-cut alias: "db")
 new         Create a new Rails application. "rails new my_app" creates a
             new application called MyApp in "./my_app"

In addition to those, there are:
 application  Generate the Rails application code
 destroy      Undo code generated with "generate" (short-cut alias: "d")
 plugin new   Generates skeleton for developing a Rails plugin
 runner       Run a piece of code in the application environment (short-cut alias: "r")

All commands can be run with -h (or --help) for more information.
```

Я подумал, что проблема в _spring_ и перезапустил его командой `spring
stop`, но ничего не изменилось.

Следующим шагом я решил прегенерить binstubs.

```
sergio@sergios-MacBook-Pro ~/Work/railsapps/tiporbet (master●)$ bundle config --delete bin                                                                                                                                                                         [ruby-2.2.0]
sergio@sergios-MacBook-Pro ~/Work/railsapps/tiporbet (master●)$ rake rails:update:bin                                                                                                                                                                              [ruby-2.2.0]
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/values/time_zone.rb:283: warning: circular argument reference - now
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/newrelic_rpm-3.8.1.221/lib/new_relic/agent/configuration/default_source.rb:674: warning: duplicated key at line 985 ignored: :disable_mongo
rake aborted!
LoadError: Please require this file from within a Capistrano recipe
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/capistrano-2.15.4/lib/capistrano/configuration/loading.rb:18:in `instance'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/rvm-capistrano-1.5.1/lib/rvm/capistrano/helpers/base.rb:16:in `rvm_with_capistrano'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/rvm-capistrano-1.5.1/lib/rvm/capistrano/helpers/_cset.rb:3:in `<top (required)>'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `block in require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:214:in `load_dependency'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/rvm-capistrano-1.5.1/lib/rvm/capistrano/base.rb:1:in `<top (required)>'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `block in require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:214:in `load_dependency'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/rvm-capistrano-1.5.1/lib/rvm/capistrano/selector.rb:1:in `<top (required)>'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `block in require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:214:in `load_dependency'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:229:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/rvm-capistrano-1.5.1/lib/rvm/capistrano.rb:3:in `<top (required)>'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:85:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:85:in `rescue in block in require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:68:in `block in require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:61:in `each'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:61:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler.rb:134:in `require'
/Users/sergio/Work/railsapps/tiporbet/config/application.rb:6:in `<top (required)>'
/Users/sergio/Work/railsapps/tiporbet/Rakefile:4:in `require'
/Users/sergio/Work/railsapps/tiporbet/Rakefile:4:in `<top (required)>'
/Users/sergio/.rvm/gems/ruby-2.2.0/bin/ruby_executable_hooks:15:in `eval'
/Users/sergio/.rvm/gems/ruby-2.2.0/bin/ruby_executable_hooks:15:in `<main>'
LoadError: cannot load such file -- rvm-capistrano
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:76:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:76:in `block (2 levels) in require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:72:in `each'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:72:in `block in require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:61:in `each'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler/runtime.rb:61:in `require'
/Users/sergio/.rvm/gems/ruby-2.2.0@global/gems/bundler-1.9.1/lib/bundler.rb:134:in `require'
/Users/sergio/Work/railsapps/tiporbet/config/application.rb:6:in `<top (required)>'
/Users/sergio/Work/railsapps/tiporbet/Rakefile:4:in `require'
/Users/sergio/Work/railsapps/tiporbet/Rakefile:4:in `<top (required)>'
/Users/sergio/.rvm/gems/ruby-2.2.0/bin/ruby_executable_hooks:15:in `eval'
/Users/sergio/.rvm/gems/ruby-2.2.0/bin/ruby_executable_hooks:15:in `<main>'
(See full trace by running task with --trace)
```

Как оказалось - это проблема с гемом _rvm-capistrano_ и началась она еще
с апдейта bundler 1.8.0, но так как у меня стояла более поздняя версия,
то все было нормально. Решением этой проблемы стало указанием `require:
false` в _Gemfile_:

```
gem 'rvm-capistrano', '1.5.1', require: false
```

Затем я нормально смог перегенерить binstubs:

```
sergio@sergios-MacBook-Pro ~/Work/railsapps/tiporbet (master●)$ rake rails:update:bin                                                                                                                                                                              [ruby-2.2.0]
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/activesupport-4.0.4/lib/active_support/values/time_zone.rb:283: warning: circular argument reference - now
/Users/sergio/.rvm/gems/ruby-2.2.0/gems/newrelic_rpm-3.8.1.221/lib/new_relic/agent/configuration/default_source.rb:674: warning: duplicated key at line 985 ignored: :disable_mongo
WARNING: Nokogiri was built against LibXML version 2.9.0, but has dynamically loaded 2.8.0
       exist  bin
   identical  bin/bundle
    conflict  bin/rails
Overwrite /Users/sergio/Work/railsapps/tiporbet/bin/rails? (enter "h" for help) [Ynaqdh] Y
       force  bin/rails
    conflict  bin/rake
Overwrite /Users/sergio/Work/railsapps/tiporbet/bin/rake? (enter "h" for help) [Ynaqdh] Y
       force  bin/rake
```

И вот, наконец-то, мое rails app запустилось!
