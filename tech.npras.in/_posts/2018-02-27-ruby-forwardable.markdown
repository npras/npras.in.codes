---
layout: post
title: "Ruby musings: Implementing Forwardable standard library"
excerpt: "Metapractice"
---

First the test cases that explain how the api is going to be:

```rb
class TestPerson < Minitest::Test
  def setup
    @address = {
      street: 'blah st',
      city: 'blah city',
      state: 'blah state'
    }
    address = OpenStruct.new(@address)
    @name = {
      first_name: 'first NAME',
      last_name: 'last NAME'
    }
    name = OpenStruct.new(@name)
    @o = Person.new address: address, name: name
  end

  def test_address_via_person
    assert_equal @address[:street], @o.street
    assert_equal @address[:city], @o.city
    assert_equal @address[:state], @o.state
  end

  def test_multiple_delegations
    assert_equal @name[:first_name], @o.first_name
    assert_equal @name[:last_name], @o.last_name
  end
end
```

Now the `Person` class that would use my `MyForwardable` module:

```rb
class Person
  attr_reader :address, :name

  extend MyForwardable
  delegate [:street, :city, :state] => :address, [:first_name, :last_name] => :name

  def initialize(address:, name:)
    @address = address
    @name = name
  end
end
```

And now, the actual module that does this functionality:

```rb
module MyForwardable
  def delegate(h)
    h.each do |del_methods, del_to|
      del_methods.each do |del_method|
        define_method del_method do
          send(del_to).send(del_method)
        end
      end
    end
  end
end
```
