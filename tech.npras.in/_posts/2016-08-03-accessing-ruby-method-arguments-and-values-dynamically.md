---
layout: post
title: Accessing Ruby Method Arguments And Values Dynamically
excerpt: Learn some voodoo magic and use it to confuse your co-coders
categories: ruby
---


Here's the magic spell:

```rb
def sample(arg1, arg2='default value2', arg3:, arg4: 'default value4', arg5: 'default value5')
  method(__method__).parameters.each_with_object({}) { |(_, key), obj|
    obj[key] = binding.local_variable_get(key).to_s
  }
end



result = sample 'value1', 'value2', arg3: 'value3', arg4: 'value4'

p result
```

This prints out all the arguments and their corresponding values as a hash:

```shell
{:arg1=>"value1", :arg2=>"value2", :arg3=>"value3", :arg4=>"value4", :arg5=>"default value5"}
```

### Breakdown and Analyze

`__method__` is a special ruby variable that returns the name of the method (as a symbol) it is placed in. So in our case, it returns `:sample`.

`method(__method__)` returns a `Method` object of this `sample` method.

`method(__method__).parameters` returns a 2D array of all the arguments. Each inner array has 2 elements: The first is the symbol representing the type of the argument. The 4 values I have seen so far are: `:req`, `:opt`, `:keyreq`, `:key`.

If you print the value of this array for the above method, you'll see this:

```
[[:req, :arg1], [:opt, :arg2], [:keyreq, :arg3], [:key, :arg4], [:key, :arg5]]
```


We are then using `each_with_object` to collect the args and their values as an key-value pair of hash. Since the type of the argument, present as the first item in each inner array, is not required for our desired output, we ignore it with `_` variable name.

To get the value of any local variable, present anywhere, we can use `binding.local_variable()`. You just have to pass the variable name as a symbol. You can try to print the value of variable 'result' using this technique:

```rb
blah= binding.local_variable_get(:result)
p blah
# outputs this:
# {:arg1=>"value1", :arg2=>"value2", :arg3=>"value3", :arg4=>"value4", :arg5=>"default value5"}
```
