---
layout: post
title: "FizzBuzz in SQL"
excerpt: "Eat this Interviewer"
---

The expected output is:

```
 x  │  value
════╪══════════
 1  │ 1
 2  │ 2
 3  │ fizz
 4  │ 4
 5  │ buzz
 6  │ fizz
 7  │ 7
 8  │ 8
 9  │ fizz
 10 │ buzz
 11 │ 11
 12 │ fizz
 13 │ 13
 14 │ 14
 15 │ fizzbuzz
 16 │ 16
 17 │ 17
 18 │ fizz
 19 │ 19
 20 │ buzz
 21 │ fizz
 22 │ 22
 23 │ 23
 24 │ fizz
 25 │ buzz
 26 │ 26
 27 │ fizz
 28 │ 28
 29 │ 29
 30 │ fizzbuzz
```

And the sql is:

```sql
select
    x::varchar,
    case
      when (x % 15 = 0) then
        'fizzbuzz'
      when (x % 3 = 0) then
        'fizz'
      when (x % 5 = 0) then
        'buzz'
      else
        x::varchar
    end as "value"
  from generate_series(1, 50) t(x)
;
```
