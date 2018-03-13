---
layout: post
title: Ruby Is Forgiving Of Spaces After Dot
excerpt: In which I show you how a mistake is not really a mistake
categories: ruby
---

You'd call methods on an object like so:

```rb
some_object.some_method
```


This is how we came to learn to see Object-Oriented Programming in practice.

I know that we can chain methods by placing each methods in a new line, like so:

```rb
res = "prasanna"
  .upcase
  .downcase
  .split('')
  .join

p res
# => "prasanna"
```

But recently I found that, there can be spaces before and/or after the 'dot' too.

Mixing this and the previous scenario:

```rb
res = "prasanna" . upcase . downcase .split('') . join
  .upcase
  .chop

p res
# => "PRASANN"
```

It works!

But remember, you'll never be lauded if you use this in a real project with collaborators!
