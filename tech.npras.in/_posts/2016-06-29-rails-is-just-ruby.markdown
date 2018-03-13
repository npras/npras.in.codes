---
layout: post
title: Rails is just Ruby
categories: rails
excerpt: where I get to quote a line from a movie that I just saw the previous day
---

>  The closer you think you are, the less you'll actually see.



It's 6am in the morning.

With a cup of strong black coffee, I sit down to write code.

Just a set of APIs and tests for them. Nothing brain-stimulating. This is right at the center of my comfort zone.

I know I'll be done in 30 minutes. It's only 5 endpoints.

I start writing the first.

I write its test.

I run it, expecting a green dot.

I get an error, not even a failure.

Some silly mistake it must be. I don't even pause to read the error message. I head out to the editor. Quickly check all 4 involved files - the routes, the controller, the view, and the test file.

No mistakes on my part.

I blindly rerun the test. Only to be stubbornly resisted with the error again.

This time I check the error message.

`@controller is nil: make sure you set it in your test's setup method.`

The fuck?

I look closer. Literally. Closer to the monitor, pupils dilated.

Couldn't find any reasons. So I willfully oblige to the error message's advice, turning a deaf ear to screaming gut.

In the test's `setup` method, I set the `@controller` instance like so:

```rb
setup do
  @controller = MyController.new
end
```

I'm only welcomed with a different error this time.

```
NoMethodError: undefined method `parameters' for nil:NilClass
```

I do have set a `@parameters` instance var in the controller. Could it be that `parameters` is a reserved keyword? I actually googled for that, found all reserved words in ruby/rails, and was left clueless again.

Now I look even closer. 2 sips of coffee to focus on 1 point without flinching.

I take out `byebug` from my arsenal.

After breaking on the very first line in my controller, `byebug` shows me what it sees.

I then press `n` hoping to walk through slowly till I find the bugger.

I'm immediately spun around and thrown away at a `rescue` section of some deeply dark pits of rails:

```rb
[665, 674] in /Users/prasanna/.gem/ruby/2.3.1/gems/actionpack-4.2.6/lib/action_controller/test_case.rb
   665:           end
   666:           unless @controller
   667:             begin
   668:               @controller = klass.new
   669:             rescue
=> 670:               warn "could not construct controller #{klass}" if $VERBOSE
   671:             end
   672:           end
   673:         end
   674:
(byebug)
```

Wat?

Printing out `$!` doesn't help:

```
(byebug) $!
#<NoMethodError: undefined method `parameters' for nil:NilClass>
(byebug)
```

I then helplessly ask Google, StackOverflow and my gut for any assistance. Nothing.

Looking up, I find more than 1 hour had passed. I still haven't got done even the first API.

I pause, give up, use the bathroom, eat things I got hold of, drink water, see some youtube videos, and did some more things I don't even remember now.

I ran the tests again hoping, against odds, against reason, that this time, somehow, the tests will pass. I bet on timezone issues.

I lost.

Then suddenly it hit me, without any prompt or whatsoever:

I needed a route called `/initialize`. And for that, I had defined an action called `initialize`.

Due to the heavy and long-lasting exposure, I've been treating the different parts of rails as first class citizens, but they are all actually normal ruby code. Just classes and modules with the rubyish bells and whistles attached.
