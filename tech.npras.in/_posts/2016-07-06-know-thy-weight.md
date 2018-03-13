---
layout: post
title: "Know Thy Object's Weight"
categories: ruby
excerpt: Be mindful of your object's memory size
---

I'm currently working on a new rails 5 API project. All response codes and response descriptions are predefined. I need to put them in namespaced constants so I can easily refer them in the code *and* in tests.

This is the interface I envisioned initially:

```rb
# some_controller.rb
def some_action
  # ...
  data = {
    code: RESPONSE::SUCCESS[:code]
    description: RESPONSE::SUCCESS[:description]
  }
  render json: data
end
```

The `RESPONSE` constant is just a container for other hash constants:

```rb
module RESPONSE
  SUCCESS = {code: '1111', description: 'success'}
  INVALID_AUTH_TOKEN = {code: '1112', description: 'invalid authtoken'}
  # ...
end
```

Immediately I desired a more object-oriented way of accessing the data. `RESPONSE::SUCCESS.code` is much better than `RESPONSE::SUCCESS[:code]`.

```rb
module RESPONSE
  SUCCESS = OpenStruct.new code: '1111', description: 'success'
  INVALID_AUTH_TOKEN = OpenStruct.new code: '1112', description: 'invalid authtoken'
  # ...
end
```

I get to type 2 chars less for each line. And I get to live longer by.. maybe a few hours?

But then a nagging feeling: I'm replacing a simple hash with a heavier OpenStruct object. In the worst case where there are huge numbers of response codes, will there be a memory problem?

[ObjectSpace::memsize_of](http://ruby-doc.org/stdlib-2.3.1/libdoc/objspace/rdoc/ObjectSpace.html#method-c-memsize_of) came to help me clarify things. You just take an object and give it to it, and it will return its size in bytes.

...and so hash vs openstruct:

```rb
require 'ostruct'
require 'objspace'

hash = {a: 'aa', b: 'bb'}
os = OpenStruct.new a: 'aa', b: 'bb'

Object.memsize_of hash
# 232

Object.memsize_of os
# 40
```

6x smaller!

Indeed! [OpenStruct objects](http://ruby-doc.org/stdlib-2.3.1/libdoc/ostruct/rdoc/OpenStruct.html) are much smaller because they don't have all the bells and whistles a [Hash object](http://ruby-doc.org/core-2.3.1/Hash.html) has.

So I had accidentally stumbled upon a better way. Is there an even better way?

```rb
RC = Struct.new :a, :b
struct = RC.new 'aa', 'bb'
ObjectSpace.memsize_of struct
# 40
```

Struct *and* OpenStruct are better than hash.

What about BasicObject?

```rb
class RC < BasicObject
  attr_reader :a, :b
  def initialize a, b
    @a = a
    @b = b
  end
end

bo = RC.new 'aa', 'bb'
bo.a   # 'aa'
bo.b   # 'bb'

ObjectSpace.memsize_of bo
# 40
```

Same as Struct and OpenStruct. So can't get any better than these? Please share if you have any ideas.
