---
layout: post
title: When to use Apply in a Javascript function
categories: javascript, ruby
excerpt: Apply is to Javascript as Splat operator is to Ruby
---

In ruby we use the 'splat' operator to capture multiple variables into an array or to spread out the contents of an array to many variables.

Capture multiple vars:

```rb
def paint!(color, *walls)
 walls.each { |w| w.paint(color) }
end

paint!('red', 'front', 'back', 'side')
```


Spread an array to multiple vars:

```rb
def do_math!(operation, num1, num2)
  num1.send(operation, num2)
end

nums = [7, 8]
do_math!(:+, *nums)
```

This second usecase is what resembles like the javascript's `apply` usecase.

You use it in place where you have an array of items that you need to pass it as args to a function, but that function doesn't take the whole array as the arg though.

So you use `Function.prototype.apply()` to `spread out` the args to the function.

Here's how it looks:

### Example 1

```js
// you have a function that adds 2 numbers:
function addTwoNums(num1, num2) {
  return (num1 + num2);
}

// you have an array of items:
nums = [1, 2]
```

You can't directly pass the array to the function:

```js
addTwoNums(nums) // will make num1 the whole array and num2 undefined
// unexpected output: "1,2undefined"
```

You need to **spread out** the array into the function. Use `apply` like so:

```js
addTwoNums.apply(null, nums)
// 3
```

Depending on your usecase, the first arg - `thisArg` - will change.

### Example 2: Another similar usage

You have `Math.max` function. It takes a list of args and returns the one with the max value. You use it like this:

```js
Math.max(1, 2, 3, 455)
// 455
```

But if you have an array of numbers, which `Math.max` clearly isn't expecting, then you spread it out using apply:

```js
nums = [1, 2, 3, 455]
Math.max.apply(null, nums)
```

### Example 3: When we want to flatten a 2D array

You have a big 2D array:

```js
matrix = [[1, 2], [3, 4], [5, 6]] // assume all the way upto 10,000

// You need this:
// [1, 2, 3, 4]
```

This one is kinda tricky.

For seemingly unapparent reason, you need to know about `Array.prototype.concat()` first, to flatten a 2D array!

```js
[].concat(1, [2, 3], [4, 5])
// [1, 2, 3, 4, 5]
```

As you can see, `concat` doesn't maintain the order of array nesting. It takes as argument another array(s) and/or primitive values. **But it doesn't work if you pass `matrix` itself as input**:

```js
[].concat(matrix)
// result would be same as matrix
```

Using these 2 `concat` quirks, you can now flatten a 2D array like so:

```js
Array.prototype.concat.apply([], matrix)
// [1, 2, 3, 4, 5, 6]
```


### Example 4: When we want to create date from string
Suppose you have a date like this:

```js
var dtstr= "2014-3-04T19:35:32Z";
```

And you want to convert it to date object using `Date.UTC`. It's signature is:

```js
Date.UTC(year, month[, day[, hour[, minute[, second[, millisecond]]]]])
```

You first split up the string date into an array that consists of the date's components:

```js
dtstr = dtstr.replace(/\D/g," ");
var dtcomps = dtstr.split(" ");
// modify month between 1 based ISO 8601 and zero based Date
dtcomps[1]--;
```

Then you use `apply` to spread out the individual components into corresponding args od `Date.UTC` function:

```js
Date.UTC.apply(null,dtcomps)
```

### Example 5: When we want to pass args from arguments to some other function

Suppose you want to write a partial function out of some other function. You might have to use `apply` there for passing args correctly.

Consider this usage: Write a generic partial function to get this functionality:

```coffee
fnAdd = (a, b) ->
  a + b

fnAdd100 = partial(fnAdd, 100)
fnadd100(14)  # 114
```

You'd go about implementing this like so:

```coffee
partial = (fn) ->
  initialArgs = Array.prototype.slice.call(arguments, 1)
  partialFn = ->
    remainingArgs = Array.prototype.slice.call(arguments)
    allArgs = initialArgs.concat(remainingArgs)
    fn.apply(this, allArgs)
  partialFn
```

## BUT!
All of this weird indirection is only because Javascript didn't have a native way to spread out items in an array like how Ruby does.

But with EcmaScript 6, that worry is no more! Check this out for the new `spread operator` and how all of the above `apply` usages can be replaced with this one:

[ES6 Spread Operator](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator)
