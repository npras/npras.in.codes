---
layout: post
title: "ES6's Generator Function and Ruby's Enumerator. What they are?"
categories: javascript, ruby
excerpt: "Learn how these two are the same, and what purpose they serve"
---

### Javascript ES6 Generator Function

[Check its docs](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/function*).

Declaring the fn:

```javascript
function* foo(){  // NOTE: the *. It declares this is as a generator fn
  var index = 0;
  while (index <= 2)
    yield index++;
}
```

Calling the iterator fn:

```javascript
var iterator = foo();  // calling it returns an iterator

iterator.next(); // { value: 0, done: false }
iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: undefined, done: true }
```

### Ruby's Enumerator

[Check its docs](http://ruby-doc.org/core-2.3.1/Enumerator.html).

Declaring the fn:

```rb
def foo
  index = -1
  yield(index += 1) while index < 2
end
```

Note: In ruby, if you simply call this function (with a block), it will behave as a normal function. You need to do this to convert this function to an enumerator:

```rb
enum = to_enum(:foo)
```

Calling the enumerator fn:

```rb
p enum.next # 0
p enum.next # 1
p enum.next # 2
p enum.next # raises StopIteration exception, saying iteration reached end
```

## Conclusion
As you can see, ES6 offers the same enumerator functionality that's already present in Ruby. If you can grasp one, you can grasp the other easily.

### What purpose does it serve?
The enumerator capability allows the caller to have the execution control of a function. We are literally saying "continue", and then after some time, "freeze!". The function stops right where it was at that point, and we can continue and play this till the end of execution!

Ruby also provides a way to rewind the enumerator back to the start once it had reached the end.

[Avdi Grimm's Rubytapas episode](http://devblog.avdi.org/2013/09/10/rubytapas-freebie-enumerator/) on this covers the enumerator's usage in more detail. Check it out, it's totally free!
