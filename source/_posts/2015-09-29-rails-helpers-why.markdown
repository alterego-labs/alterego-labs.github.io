---
layout: post
title: "Rails Helpers: WHY?!?"
date: 2015-09-29 14:10:29 +0300
comments: true
author: Sergey Gernyak
categories: [rails, helpers]
---

Наверное самым странным и непонятным (по моему мнению) компонентом
Rails являются хелперы (Helpers). Когда мы приходим в мир Rails мы много
читаем про MVC и как она реализована в Rails, прозоны ответственности
этих компонентов. Но что представляют собой Helpers, которые содержатся
в папке `app/helpers`?

### Что предгагают нам рельсы

Хелперы представляют собой обычные модули с набором функций. Например:

```ruby
module ApplicationHelper
  def some_useful_method(arg1)
    # some logic here
  end
end
```

Модули не являются самостоятельными единицами и в практически всегда они
инклудятся в определенный контекст для расширения.

Хелперы придумали для выноса части различной мелкой (в некоторых случаях
не такой уж и мелкой) логики из контролера и представления (view).
Например:

```ruby app/helpers/application_helper.rb
module ApplicationHelper
  def hello_text
    'Hello from Helper!'
  end
end
```

```ruby app/controllers/welcome_controller.rb
class WelcomeController < ApplicationController
  def index
    # here `hello_text` will be available
  end
end
```

```erb app/views/welcome/index.html.erb
<%= hello_text %>
```

### Какие проблемы

Основная проблема в том, что хелперы превращаются в неконтролируемую
свалку ни к чему не привязанных функций, ведь по умолчанию все хелперы
инклудятся в любой контроллер. При этом возникает вероятность
существования нескольких функций с одинаковыми именами и тогда удачной
отладки. До определенного момента в контроллер инклудился хелпер имя
модуля которого является часть имени контроллера без _Controller_ части.

Так как хелперы инклудятся в контекст view, то в них доступны instance variables объявленные в
контроллере. Поэтому это уменьшает возможность повторного использования,
а также увеличивает непонимание того, какие объекты являются "игроками"
в той или иной функции, и возвожность протестировать. Например:

```ruby app/helpers/application_helper.rb
module ApplicationHelper
  def build_push_state(except: [], with: {})
    @push_state = PushStateBuilder.new(params, except: except, with: with).build
  end
end
```

Также из-за особенности, что хелперы доступны и в контроллере и в
представлении, иногда трудно сказать в каком логическом контексте текущая функция
должна вызываться, хотя она может быть универсальной и вызываться в
обеих.

Очень часто хелперы используют для формирования html сниппетов:

```ruby app/helpers/application_helper.rb
module ApplicationHelper
  def error_messages_for(resource, decorated = true)
    return "" if resource.errors.empty?
    errors_text = resource.errors.full_messages.map { |msg| content_tag(:li, msg) }.join
    error_messages errors_text, decorated
  end
end
```

Мне кажется, что формирование HTML кода необходимо оставлять на view.

__Резюмирую__: в итоге получается модули, в которых масса отдельно
сущестующих методов, которые привязаны к разным логическим контекстам выполнения
(controller или view), которые могут генерировать HTML код и много всего
другого, чего они не должны делать.

### Как уменьшить боль?

Я бы не рекомендовал использовать хелперы вообще. Но бывают ситуации,
особенно на старте проекта, когда нужно быстро накидать рабочий
прототип, при этом особо не заморачиваясь с продумыванием сложной
объектной иерархии с распределением обязанностей. Поэтому я выделю
несколько маленьких рецептов:

1. Не использовать instance variables, объявленные в контроллере или во
   view. Передавайте явно параметры. При этом можно избежать side
   effects, когда вы вызываете функцию, а там используется instance
   variable, которой еще нет. В идеале функции, объявленные в хелперах,
   должны быть [pure functions](https://en.wikipedia.org/wiki/Pure_function).

2. Выносите формирование HTML сниппетов в отдельный partial. Опять же
   бывают ситуации, когда сниппет - это буквально одна строчка,
   например, в зависимости от статуса выводить разную иконку.

3. Отказаться от глобального инклудинга всех хелперов. Для этого
   необходимо добавить следующую строку, например в
   `config/application.rb`:

   ```ruby
   config.action_controller.include_all_helpers = false
   ```
4. Отказаться от хелперов и посмотреть в сторону __View Object__ или
   __ViewModel__ :-)

### Полезные ссылки

- [https://github.com/rails/rails/blob/4-2-stable/actionpack/lib/action_controller/metal/helpers.rb](https://github.com/rails/rails/blob/4-2-stable/actionpack/lib/action_controller/metal/helpers.rb)
- [https://en.wikipedia.org/wiki/Pure_function](https://en.wikipedia.org/wiki/Pure_function)
- [https://github.com/apotonick/cells](https://github.com/apotonick/cells)
- [https://github.com/drapergem/draper](https://github.com/drapergem/draper)
