---
layout: post
title: "Ruby метапрограмминг и наследование"
date: 2015-05-25 11:23:10 +0300
comments: true
author: Sergey Gernyak
categories: [ruby, metaprogramming, inheritance, fun]
---

Ruby не перестает меня удивлять! Как по мне, то самое удивительное и
мощное в нем - метапрограмминг. После статического языка, в котором твой
арсенал ограничен заложенными ключевыми словами и конструкциями, просто
невозможно вообразить насколько развязаны твои руки и что ты можешь со
всем этим делать! Используя динамику и выразительность Ruby можно
превратить кусок кода практически в понятную фразу на английском.

Элементы конструкций не ограничены! Можно создавать их самому сколько
угодно, получая в результате высокоуровневый и понятный почти любому
человеку DSL.

Самое главное вовремя остановится... Но не сейчас)

### Варианты объявления класса

Все мы привыкли объявлять наши классы используя элемент синтаксиса ruby,
а именно ключевое слово __class__:

``` ruby
class User
end
```

Но существует еще один вариант. Т.к. класс является экземпляром класса
Class, то справедлива следующая запись:

```ruby
User = Class.new do
  attr_accessor :name
end
```

Используя способ выше также можно указать суперкласс, просто передав его
как параметр в метод `Class.new`:

```ruby
class User
end

RegisteredUser = Class.new(User) do
end

RegisteredUser.ancestors # => [RegisteredUser, User, Object, Kernel, BasicObject]
```

__Но есть одна интересная особенность!__

Когда вы несколько раз объявляете класс используя ключевое слово _class_, то реально
класс инициализируется только один раз. Все следующие разы класс
__открывается__ для внесения изменений. Например:

```ruby
class User
end

User.object_id # => 70299452133440

User.new.name # => NoMethodError: undefined method `name' for #<User:0x007fd35b40a930>

class User
  def name
    'Sergey'
  end
end

User.obect_id # => 70299452133440

User.new.name # => Sergey
```

В случае с объявлением класса, используя метод `Class.new`, ситуация
немного другая. На самом деле слово `User`, которое мы используем для
инициализации экземпляров, является практически такой же переменной,
которая ссылается на экземпляр класса `Class`. Поэтому код ниже дает,
может быть, вполне очевидные результаты:

```ruby
class User
  def name
    'Sergey'
  end
end

User.object_id # => 70162158343080

User.new.name # => Sergey

User = Class.new do
  def second_name
    'Gernyak'
  end

  def full_name
    "#{second_name} #{name}"
  end
end

User.object_id #=> 70162144228540

User.new.full_name # => NameError: undefined local variable or method `name' for #<User:0x007fdb3b028b18>

User.new.second_name # => Gernyak
```

Получается мы взяли и заменили значение нашей переменной `User` на
другое. Об этом также свидетельствует значение `object_id`. Поэтому наш
новый класс `User` ничего не знает о методе `#name`.

Правда и тут можно сделать хитрость: добавить наш существующий класс в
цепочку наследования:

```ruby
class User
  def first_name
    'Sergey'
  end
end

User.new.name # => Sergey

User = Class.new(User) do # <= Вот сюда
  def second_name
    'Gernyak'
  end

  def full_name
    "#{second_name} #{first_name}"
  end
end

User.new.full_name # => Gernyak Sergey

User.ancestors # => [User, User, Object, Kernel, BasicObject]
```

### Немного магии

А почему бы не объявить динамически класс, как суперкласс для другого
класса?

```ruby
class User < Class.new
end

User.ancestors # => [User, #<Class:0x007fd241dae998>, Object, Kernel, BasicObject]
```

Данную фичу я использовал в одной из наших библиотек [exracted_validator](https://github.com/alterego-labs/extracted_validator).

У меня есть некоторый базовый класс, который содержит общую логику для
возможности вынесения валидаций из модели:

```ruby
module ExtractedValidator
  class Base < SimpleDelegator
    # Some logic here
  end
end
```

Но с некоторыми видами валидаций могут возникнуть проблемы, т.к. они
жестко завязаны на именование класса, в котором они объявлены. Например,
валидация __uniqueness__. Для того, чтобы иметь возможность использовать
такую валидацию в кастомном валидаторе, необходимо переопределить метод
`.model_class`:

```ruby
class SignUpUserValidator < ExtractedValidator::Base
  def self.model_class
    User
  end
end
```

Используя последний код, все будет работать как и ожидается и все
останутся довольны.

Ну почти все... Я не доволен тем, что мне принудительно нужно добавлять
еще один метод каждый раз, когда буду объявлять новый валидатор. Поэтому
я решил сделать небольшой рефакторинг используя метапрограмминг :-)

И вот что получилось:

```ruby
module ExtractedValidator
  class Base < SimpleDelegator
    def self.[](model)
      Class.new(self) do
        define_singleton_method :model_class do
          model
        end
      end
    end
  end
end

class SignUpUserValidator < ExtractedValidator::Base[User]
end
```

В данном примере динамически создается класс-обертка, в котором
объявляется требуемый метод `.model_class` и этот метод возвращает
переданный класс, целевой для данного валидатора, модели. Цепочка
наследования может выглядеть следующим образом:

```ruby
SignUpUserValidator.ancestors # => [SignUpUserValidator, #<Class:0x007f8ec9b604b8>, ExtractedValidator::Base, SimpleDelegator, Delegator, #<Module:0x007f8ec9c1a138>, BasicObject]
```

Вот `#<Class:0x007f8ec9b604b8>` как раз и есть тот самый класс-обертка!


На этом у меня все. __Спасибо за внимание!__
