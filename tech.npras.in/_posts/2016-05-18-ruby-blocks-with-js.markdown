---
layout: post
title: Understanding Ruby Blocks with the help of Javascript
categories: code
excerpt: A new perspective to understand Ruby blocks
---

Does the internet really need another post explaining ruby blocks? While you are pondering this... A comment I made in Avdi Grimm's post about the customary ruby-block-explaining-blogpost struck a chord with him so much that he cited it inline in the post. In all vanity, here's the post for posterity: [http://devblog.avdi.org/2015/01/16/why-does-ruby-have-blocks/](http://devblog.avdi.org/2015/01/16/why-does-ruby-have-blocks/) From that moment on, a blog detailing the remark in Avdi's post became imminent. I was like, "Yeah, let me use this as a reason to b-l-o-g". The post, ladies and gentlemen, is here. (slow awkward applause) So... Ruby blocks. They are everything that gets said about them.

*   They are the code between the `do` `end` blocks. They are the code between `{` and `}`
*   They are closures
*   They are cool, but confusing
*   They go by 2 kind: procs and lambdas

What doesn't get said about them, though, is this: **Ruby blocks are basically just ruby methods.** 

Let that sink in.

You can do with it what you can do with normal ruby methods. You can design it to take zero or more arguments, make some of them optional, return a result expression, call it etc. Blocks are just convenient constructs introduced by Matz to reduce the verbosity unlike in other languages. It's not even a unique feature of Ruby. Most programming languages have a notion of anonymous functions. Ruby's blocks are exactly that. Anonymous == The Function With No Name, if you know what I mean. Take Javascript for example. Most webdevs are familiar with jquery and would instantly recognize this idiomatic way of defining a handler for an element. Here's a cliche straight out of the docs:

```js
$( "#target" ).click(function() {
alert( "Handler for .click() called." );
})
```

  In boring details, what happens here is that the `click method` on the jquery collection takes exactly one argument - the `clickHandler function`. What the `click method` does with that is - yes, boring - it calls it when the click event is triggered on that element. Something like that in Ruby will look like this:  

```rb
# Note: Ruby doesn't implement DOM manipulations like jquery.
# That's not even a thing now! Nor will it ever be.
# Amen.

jq_selector("#target").click do
  puts "Handler for .click() called"
end
```

So if you compare these 2, you'll see the similarity between the function that we passed in javascript and the "do..end" block we passed in ruby. Now, how to show that the block is actually another ruby method? Before that, a small detour. These examples are not practical. You won't have the gratification of executing them and seeing the results quickly without having to resort to write some slightly difficult setup code. So, let's write a more practical examples that are similar to the above snippets. These JS examples can be run right within your browser's javascript console, and the ruby examples can be run in irb (or in a file and run it with `ruby` command). The JS version:

```js
// defining the 'greet' fn, which can be seen as something similar to the 'click' fn above:
function greet(personFn){
  return "hello " + personFn();
}
```

```js
// invoking (aka calling) the fn:
greet(function(){
  return "Joe";
});
```

  The exact same code in Ruby:

```rb
# The fn definition
def greet(&blk)
  "hello #{blk.call}"
end

# The fn invocation
greet do
  "Joe"
end
```

    In both cases, we can say that there's a function `greet` that takes a method as an argument and returns a "hello" string prepended with the result of invoking the passed-in method. The above js and ruby snippets can be re-written with clarity as follows. Before actually invoking the `greet` function, we just pull out its argument into a variable and pass the variable instead. Here it is: The js version:

```js
var personFn = function(){
  return "Joe";
};

// invoking by passing the above variable
greet(personFn) // result: "hello Joe"</pre>
```

  The corresponding ruby version:

```rb
person_fn = lambda do
  "Joe"
end

greet(&person_fn) # result: "hello Joe"
```

### Passing arguments to the block

Since we've seen that blocks are no different than the standard ruby methods, its only natural to expect them to take arguments. We'll see a standalone example.

```rb
sum_blk = lamdba(x, y) do
  x + y
end

# invoking it
sum_blk.call(4, 8) # outputs 12
```

### Syntax Quirk

You might have noticed a difference between the js and ruby version here though, all along the previous examples. While we invoke a js function passed into another function in the standard way - brackets at the end of the name: `personFn()` - in ruby though, we call it with `person_fn.call`. I don't have an explanation as to why this is the case, but I do think you should remember this distinction! Also there are a few other ways to invoke a block other than using `call`:

```rb
say_hi = lambda do |name|
  "hi #{name}"
end

say_hi.call('pras') # 'hi pras'
say_hi.('pras') # 'hi pras'
say_hi['pras'] # 'hi pras'
say_hi === 'pras' # 'hi pras'
```

  Wow! I have only one question Matz. "_But Why?_" (Note: These various forms have their own place and feel surprisingly natural when seen there. Matz is beyond your comprehension. Eg: See this to know what craziness means: [Programming with Nothing](https://codon.com/programming-with-nothing))  

### Fun Thought

What if Matz had implemented closures, exactly the same way as in javascript? No special keywords like 'do..end'. No special invokation syntax like `call`. No weird way of defining arguments between pipes (?!!). How would ruby blocks look then?

```rb
# invoking the greet function.... by passing in another function 'literally'
greet def
  'Joe'
end

# passing a function that can take arguments on its own?
greet def greeted_name, salutation
  'Joe'
end
```

  That's not mind-bending, but - but - yes, that's the word - that's weird. But only because we are not accustomed to it. Who knows, we might've got used to this if it had become the actual implementation.

### What about differences between Procs and Lambdas

Yes there are significant differences wrt extra-parameter passing, the `return` statement etc. But they are not used for these differences, but for their closure behaviour. Whenever in doubt, just use lambda as it's behaviour is more inline with the natural method. You can read about the differences here: [http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/](http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/) So, there goes it. A blog post off my chest.
