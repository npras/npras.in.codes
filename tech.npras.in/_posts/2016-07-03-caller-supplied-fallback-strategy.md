---
layout: post
title: Caller-supplied Fallback Strategy
categories: ruby
excerpt: Inject Failure-handling code as a dependency!
---

I'm reading the exceptional Avdi Grimm's [Exceptional Ruby](http://exceptionalruby.com/) book. (Thanks Cass!)

Like Avdi and Ruby, the book is also exceptional!

One section of the book is **"Your Failure handling Strategy"**. It has a handful of advices, one of which is this: *Caller-supplied fallback strategy*.

Last week at work, I stumbled upon a problem. Within minutes I had the solution too. I wrote test cases, wrote the solution code and pushed and closed the story when tests were green.

But it felt wrong to me. I wasn't convinced of my solution. I knew there had to be a better way, but I couldn't come up with an elegant replacement.

Until now, after reading this chapter in the book.

First, the problem, and the original solution I had conceived.

### Problem statement

I want all the ineffective cats of a user. (Don't ask me what I'm going to do with them).

User has many cats.

A Cat is effective when it has both teeth and claws (or nails?).

A Cat is ineffective when it has only one of these 2.

For the sake of the example, assume that humans are fair people and don't consider a cat ineffective if it doesn't have both.

It's a fair comparison right? If you compare a weaponless cat with a weaponised cat, then you are a... catist! Don't be a catist, be a fair human. OK?

So here's the `cat.rb`:

```rb
def effective?
  if teeth && claws
    return true
  elsif teeth or claws
    return false
  else
    fail "You Catist!"
  end
end
```

My Design rationale: the `cat.effective?` method can only return true or false. Anything else it is asked to do that it doesn't know, it just gives up by raising an exception.

And at `user.rb`, I collect all ineffective cats like so:

```rb
def ineffective_cats
  self.cats.reject do |cat|
    cat.effective? rescue true
  end
end
```

That line with `rescue true` is what made me uncomfortable. I had to add it because that's the only way the cats with neither weapons will not be included in the `ineffective_cats` list.


### Enter Exceptional Ruby
One of the gems from the book is this:

> Exceptions shouldn’t be expected. Use exceptions only for exceptional situations.It is hardly exceptional to fail to open a file.

> When writing an application you expect invalid input from users. Since we expect invalid input we should NOT be handling it via exceptions because exceptions should only be used for un-expected situations.

Light-bulb moment! So a cat with no weapons is totally expected inside of the `cat.effective?` method! It's not an abnormality. It's not an exception. It's just a different part of the rule.

So, what can be returned in the place of the `fail` line? The truth obviously. If you ask a weaponless cat if it is effective or not, it can't say true or false. Technically it can only say 'nothing', aka `nil`.

So, `cat.rb` becomes:

```rb
def effective?
  if teeth && claws
    return true
  elsif teeth or claws
    return false
  else
    nil # NOTE
  end
end
```

Now, another gem from the book:

> In most cases, the caller should determine how to handle an error, not the callee.

So it's now obvious that the caller has to decide what to do if one of the cats returned `nil` when asked if effective.

Well, for your usecase, we can simply ignore the cat that returned `nil` from the final list of ineffective cats. Like so:

```rb
# user.rb
def ineffective_cats
  self.cats.reject do |cat|
    result = cat.effective?
    (result.nil? || result) ? true : false
  end
end
```

This can be expressed more succintly as:

```rb
def ineffective_cats
  self.cats.reject do |cat|
    cat.effective? or true
  end
end
```

This appears to be only slightly better than our initial version where we used `recue true`. But we have deliberately used 2 of the approaches described earlier here from the book:

* For the case of a weaponless cat, instead of raising an exception, we are returning nil. A weaponless cat is not unexpected in that method, so no need for exceptions
* We have let the caller decide what to do with the `nil` return value. Our caller had various options: It could have raised hell, it could have logged a warning saying there's a weaponless cat in the system, or it could have sneakily send it along with the weaponised cats. But our caller chose to simply ignore the weaponless cats, because it should!

This post still doesn't use "caller supplied fallback strategy" in its proper form. I'll do it here:

```rb
# cat.rb
def effective?(failure_policy=method(:raise))
  if teeth && claws
    return true
  elsif teeth or claws
    return false
  else
    fail "You Catist! This cat has no weapons, you can't consider it weaponsgrade!"
  end
rescue => e
  # we are 'punting' the decision to act on the exception up to the caller.
  failure_policy.call(e.message)
end
```

```rb
# user.rb
def ineffective_cats
  failure_policy = ->(exception) do # NOTICE: we don't do anything with the exception
    warn "A weaponless cat found. Just thought you should know."
    true
  end
  self.cats.reject do |cat|
    cat.effective?(failure_policy)
  end
end
```

This is "Caller **supplied** fallback strategy" in a nutshell.
