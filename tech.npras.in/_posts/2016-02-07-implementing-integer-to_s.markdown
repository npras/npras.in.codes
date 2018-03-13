---
layout: post
title: Implementing Integer#to_s naively
categories: code
excerpt: 1.to_s != too_easily.implemented!
---

The latest episode in Avdi Grimm's excellent Rubytapas is "381\. Integer to String". In it, he tries to implement the `123.to_s` functionality, which returns a string `"123"`. However in this episode he only solves conversion upto 2 digits of input, and has promised to implement the full version in the next episode. I thought I had about 30 minutes and started trying. My method `my_to_s` successfully converts any length of input integer to the corresponding string value. It also prepends a minus sign if the input is a negative value. However it falls flat in its nose when the input is a float. The native implementation works like a charm:

```rb
45.345.to_s  #  "45.345"
```

  Anyway, I'll have to find time to sit and think through this, or probably wait for Avdi's next episode where he unravels the "thrilling climax" of coding this functionality in a proper algorithmic way. Here's my code for your musing in the mean while.

```rb
def my_to_s(num)
  result = ''
  codepoints = {
    0b0000 => 48, # 0
    0b0001 => 49, # 1
    0b0010 => 50, # 2
    0b0011 => 51, # 3
    0b0100 => 52, # 4
    0b0101 => 53, # 5
    0b0110 => 54, # 6
    0b0111 => 55, # 7
    0b1000 => 56, # 8
    0b1001 => 57, # 9
  }
  negative = num < 0
  q = negative ? -num : num
  loop do
    q, r = q.divmod(10)
    if q < 10
      result.prepend(codepoints[r].chr)
      result.prepend(codepoints[q].chr)
      break
    else
      result.prepend(codepoints[r].chr)
    end
  end
  result.prepend(?-) if negative
  result
end

#num = 1
#num = 0b0111 # 7
#num = 0b1010 # 10
#num = 0b1111011 # 123
num = -0b10101000110000000 # 86400
p my_to_s num
```

This is the "draft" version of the code. It's not refactored, and I don't intend to considering the amount of pending tasks for the day.
