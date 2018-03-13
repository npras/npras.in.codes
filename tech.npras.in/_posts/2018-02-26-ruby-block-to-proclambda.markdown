---
layout: post
title: "Ruby musings: Converting a block to a proc/lambda"
excerpt: "Yet another customary ruby post about blocks"
---

A block is not an object (rare exception in ruby). A proc/lambda is an object.

A block can only be called using `yield`.

A proc can be called using `blk.call`.

`&` is used to convert a block to a proc and viceversa.

Block to proc:

```rb
# '&' in method definition converts the passed block to proc obj
def block_to_proc(name, &blk)
  blk.call name
end

res = block_to_proc('pras') do |name|
  "block to proc! hello #{name}"
end
p res
```

and proc to block:

```rb
def proc_to_block(name)
  yield(name)
end

my_proc = ->(name) do
  "proc to block! hello #{name}"
end
# '&' in method call converts the proc obj to a block, which is accessed
# within the method using 'yield'
res = proc_to_block 'pras', &my_proc
p res
```

### Pass around blocks or procs from method calls to method definitions without the `&` woodoo

Block used as block:

```rb
# the passed block is directly accessible via yield
def block_to_block(name)
  yield name
end

res = block_to_block('pras') do |name|
  "block to block! hello #{name}"
end
p res
```

Proc used as proc:

```rb
# the passed proc obj can be accessed in a var just like other objs
def proc_to_proc(name, blk)
  blk.call name
end

my_proc = ->(name) do
  "proc to proc! hello #{name}"
end
res = proc_to_proc 'pras', my_proc
p res

```
