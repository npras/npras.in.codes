---
layout: post
title: "Advanced Regular Expressions"
excerpt: "A typical Code Ninja's meaning of a slick weapon is regex. Learn it."
---

## Capture Group
Swap last name and first name.

```rb
s = "Prasanna Natarajan"
changed = s.gsub /(\w+) (\w+)/, '\2 \1'
```


Use this capturing to refer parts in regex.

```rb
r = /(\w)\w\1/
# matches words like 'regex', 'paras', 'madam'
```

To change this: `16 + 123` to `16 + 123 = 123 + 16`:
```rb
s.gsub /(\d+) \+ (\d+)/, '\1 + \2 = \2 + \1'
```

Non-capturing group `(?:...)`:
```rb
s = '123 + 456'
s.gsub /(\d+) \+ (?:\d+)/, '\1 + \2 = \2 + \1'
# "123 +  =  + 123"
```

To match either `tap` or `top`: `/t[ao]p/`. (I incorrectly thought `/ta|op/`. While `/t(a|o)p/` could work, the capture group is unnecesary.)


## Lookaheads & Lookbehinds
Lookarounds (both ahead and behind) are 0-width assertions. It means that they don't consume any character when matching the chars following them. It's like "having a peek ahead", but not actually moving the cursor forward/behind.

This allows us to add regex conditions for the chars we "looked around" too, without throwing them away.


### Positive Lookahead `(?=)`

Split a word into an array based on capital letters:
```rb
# this results in unexpected values in array.
"ScheduledEvent".split /[A-Z]/
# [ "", "cheduled", "vent" ]

# but we can use lookahead to keep the upcase letters:
"ScheduledEvent".split /(?=[A-Z])/
# ["Scheduled", "Event"]
```

Make sure the string has first 1 or at most 2 alphabets in the beginning, and 2 or at most 3 numbers in the end. But also make sure the whole match is exactly 4 chars long.
```rb
r = /
(?=[a-z0-9]{4}) # the lookahead part. Just ensures length is 4
[a-z]{1,2}\d{2,3} # the actual check
/x  # free-spacing mode. allows to split the regex into multiple lines and add useful comments

'ab12'.match r
'a123'.match r
'asdf23sad'.match r
```


### Negative Lookahead `(?!)`

Just the opposite of above. Matches only if the lookahead regex is NOT present.

Suppose in this kind of strings, you want an array of quantity of all fruits except that of apple: `5kg of apple, 6kg of orange, 7kg of grapes`.
```rb
r = /(\d+)kg of (?!apple)/
"5kg of apple, 6kg of orange, 7kg of grapes".match(r)
# [6, 7]

# works in any order!
"6kg of orange, 7kg of grapes, 5kg of apple".match(r)
# [6, 7]
```


###  Positive Lookbehind `(?<=)`

Find 'x' that's preceded by 'y'.
```rb
"99 books for USD100".match(/(?<=USD)\d+/) # matches 100
"thingamabob".match(/(?<=a)b/) # matches the first 'b'
```

###  Negative Lookbehind `(?<!)`

Find 'x' that's NOT preceded by 'y'.

Find all words in a sentence that doesn't end with 's':
```rb
"asdf asdds 222 111s".match(/\b(\w+)(?<!s)\b/) # [asdf, 222]
```
__Pause__ for a moment to admire the beauty of this! Look how the end of the word is captured using word boundary!


## Combining the lookarounds
Suppose there's a text box in a web page we are building that takes in some amount of money as its value. We want to add a thousandth separator automatically as the user types. How would we do that?

The regex to identify the digit that has exactly 3 digits (and no more) following it, can be found at rubular: http://rubular.com/r/0c6RqOHkHR

```js
var regex = /\d(?=\d{3}(?!\d))/
```
For input '1234', this matches '4'. For input '12345', this matches '2'.

So, to add the thousandth separator, we need to use `replace` and capture the matching digit, so that we can add a comma after it.

```js
value.replace(/(\d)(?=\d{3}(?!\d))/g, "$1,");
```
