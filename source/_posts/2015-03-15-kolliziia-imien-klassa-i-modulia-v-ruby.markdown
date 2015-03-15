---
layout: post
title: "Коллизия имен класса и модуля в Ruby"
date: 2015-03-15 09:33:25 +0200
comments: true
author: Sergey Gernyak
categories: [ruby, object model, metaprogramming]
---

### Вот такая вот ситуация

В основе статьи лежит ситуация, с которой я и мои коллеги столкнулись
при работе над одним проектом.

Представим себе следующее. У меня есть модель `Item`:

{% codeblock lang:ruby Листинг 0 - Модель Item app/models/item.rb%}
class Item < ActiveModel::Base
end
{% endcodeblock %}

Затем нам дали задачу на новую фичу: пользователи могут создавать списки
и добавлять туда элементы. Мы сразу видим модель `List`:

{% codeblock lang:ruby Листинг 1 - Модель List app/models/list.rb %}
class List < ActiveModel::Base
end
{% endcodeblock %}

Но для хранения связи между элементом и листом мы решили использовать модель
не `ListItem`, а положить ее в namespace `List`, т.е. логически
выделить эту часть подсистемы, т.к. мы предполагаем, что будут еще связанные модели.
И получается следующее:

{% codeblock lang:ruby Листинг 2 - Модель List::Item app/models/list/item.rb %}
module List
  class Item < ActiveModel::Base
  end
end
{% endcodeblock %}

Как бы все идет хорошо до тех пор, пока вы не захотите использовать
`List::Item` модель где-нибудь в коде приложения:

{% codeblock lang:bash Листинг 3 - Использование List::Item в rails console %}
Loading development environment (Rails 4.0.4)
2.1.2 :001 > List::Item.new
TypeError: List is not a module
	from /Users/sergio/Work/railsapps/namespace_collision_demo/app/models/list/item.rb:1:in `<top (required)>'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:424:in `load'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:424:in `block in load_file'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:616:in `new_constants_in'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:423:in `load_file'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:324:in `require_or_load'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:463:in `load_missing_constant'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/activesupport-4.0.4/lib/active_support/dependencies.rb:184:in `const_missing'
	from (irb):1
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/railties-4.0.4/lib/rails/commands/console.rb:90:in `start'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/railties-4.0.4/lib/rails/commands/console.rb:9:in `start'
	from /Users/sergio/.rvm/gems/ruby-2.1.2/gems/railties-4.0.4/lib/rails/commands.rb:62:in `<top (required)>'
	from bin/rails:4:in `require'
	from bin/rails:4:in `<main>'
{% endcodeblock %}

Есть несколько путей решения этой проблемы.

### Решение 0 - Внести название модуля в название класса

Я до конца не понимаю почему, но вот если изменить код модели
`List::Item` на следующий:

{% codeblock lang:ruby Листинг 4 - Решение 0 для модели List::Item app/models/list/item.rb %}
class List::Item < ActiveModel::Base
end
{% endcodeblock %}

то все рабоет нормально:

{% codeblock lang:bash Листинг 5 - Результат решения 0 %}
2.1.2 :001 > List::Item.new
 => #<List::Item id: nil, created_at: nil, updated_at: nil>
{% endcodeblock %}

### Решение 1 - Заменить module на класс

Изучая объектную модель ruby мы обнаружили одну интересную вещь: класс -
это тот же модуль, но с дополнительным функционалом!

{% codeblock lang:ruby Листинг 6 - Отношения Class и Module %}
2.1.2 :013 > List::Item.class
 => Class
2.1.2 :014 > List::Item.class.superclass
 => Module
{% endcodeblock %}

Значит `Class` наследует весь функционал `Module` и он также может
выступать как контейнер. Давайте попробуем это.

{% codeblock lang:ruby Листинг 7 - Решение 1 для модели List::Item app/models/list/item.rb %}
class List
  class Item < ActiveModel::Base
  end
end
{% endcodeblock %}

И правда - все работает отлично:

{% codeblock lang:bash Листинг 8 - Результат решения 1 %}
2.1.2 :001 > List::Item.new
 => #<List::Item id: nil, created_at: nil, updated_at: nil>
{% endcodeblock %}

### Решение 2 - К черту всю эту возню с объектной моделью!

В данном случае предлагается поступить следующим образом: выделить
namespace и ложить туда все модели, которые логически связаны. В
соответствии с нашим примером будет следующее:

{% codeblock lang:ruby Листинг 9 - Модель Lists::List app/models/lists/list.rb %}
module Lists
  class List < ActiveModel::Base
  end
end
{% endcodeblock %}

{% codeblock lang:ruby Листинг 10 - Модель Lists::Item app/models/lists/item.rb %}
module Lists
  class Item < ActiveModel::Base
  end
end
{% endcodeblock %}

Однозначным плюсом данного подхода является то, что все модели лежат в
одной папке и если вам нужно выпилить данный кусок функционала, то вот
оно все в папке лежит - не нужно шарится среди несколько десятков
моделей в папке `app/models`.
