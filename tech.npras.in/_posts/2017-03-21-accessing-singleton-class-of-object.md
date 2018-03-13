---
layout: post
title: "CodeRead: metaclass gem: Accessing An Object's Singleton Class"
excerpt: "Easily gain access to an object's secret room"
---

In ruby, an object is responsible for holding it's data attributes.

```rb
o = Object.new
o.instance_variable_set "@name", "box"
o # #<Object:0x007fbc7912d5a8 @name="box">
```

But it's not responsible for holding it's methods. An object's methods reside in its defining class. And if an object has a specific method just for itself, it's held in the object's singleton class.

```rb
class Dummy
  def say_hello
    'hello'
  end
end

d = Dummy.new
p d.methods.grep /say_hello/  # [:say_hello]
p Dummy.instance_methods.grep /say_hello/  # [:say_hello]

sc = class << d; self; end


sc.class_eval { def say_hi; 'hi'; end }

p d.say_hi  # 'hi'

p d.methods.grep /say_hi/  # [:say_hi']
p Dummy.instance_methods.grep /say_hi/  # []
```

In the last line, note that the `say_hi` method isn't seen in `Dummy`'s list of instance methods.


### The `metaclass` gem

This [gem](https://github.com/floehopper/metaclass) gives easy access to an object's metaclass (same as singleton class) through a `__metaclass__` method. It's implementation is easy.

```rb
Dummy = Class.new do
  def __metaclass__
    class << self
      self
    end
  end
end

obj = Dummy.new


obj.__metaclass__.class_eval { def only_me; 'Yes You'; end }


obj2 = Dummy.new

p obj.only_me # 'Yes You'
```
