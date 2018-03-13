---
layout: post
title: Implementing attr_accessor using define_method
categories: code
excerpt: An exercise in metaprogramming
---

This is a pretty common exercise that all ruby beginner's go through when trying to understand Ruby.

<a href="https://www.google.com/?gws_rd=ssl#q=implementing+attr_accessor+:ruby">The internet is already filled with blog posts on how to do this</a>. But they use `class_eval`. This customary blog post will use `define_method` to do the same. I'm 'completing' the internet y'know.

```rb
module Kernel
  def my_attr_accessor *attributes
    attributes.each do |attr|
      define_method "#{attr.to_s}" do
        instance_variable_get "@#{attr.to_s}"
      end
      define_method "#{attr.to_s}=" do |value|
        instance_variable_set "@#{attr.to_s}", value
      end
    end
  end
end

class Person
  my_attr_accessor :name, :age
end

john = Person.new

john.name = 'John'
john.age = 22
puts john.name
puts john.age

john.name = 'BigJohn'
john.age = 30
puts john.name
puts john.age
```
