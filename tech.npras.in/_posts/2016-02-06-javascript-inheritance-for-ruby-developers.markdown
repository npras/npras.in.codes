---
layout: post
title: Javascript Inheritance for Ruby Developers
categories: code
excerpt: Comparing can lead to learning even better
---

In Ruby we have first-class syntax support to do almost anything required to do Object Oriented Programming, or even any other facets of programming techniques. We have procs, lambdas, inheritance, ability to include or extend a module, class and object concepts etc. That's why it is attractive as we have concise syntax for almost everything we'd ever want.

But in Javascript there are only a very few of these. No special syntax support for defining classes, and no straight-forward inheritance support. All it has is these: a well defined object and function entities, and infallible concepts like prototype, object binding, scopes and contexts.

However, with these minimal capabilities, and with a thorough grasp of the language's strengths and weaknesses, you can do almost anything with Javascript. In the face of emerging front-end frameworks and NodeJS, it's high time you get involved in understanding and mastering the Javascript. In this post, we'll see how we can achieve inheritance in Javascript by juxtaposing inheritance from Ruby.

### Inheritance

What is inheritance in Object Oriented Programming? I can come up with 3 minimal tests to decide if inheritance is implemented or not.

*   A Subtype object should be and instance of both the Subtype and the Supertype from which Subtype inherits.
*   The Subtype object should inherit properties from the Supertype definition.
*   Subtype should be able to override properties defined in Supertype.

We'll see examples of these ideas using Ruby.

### Ruby's Inheritance

Consider a car object which is of a specific make - Hyundai I20Asta. It can have make-specific properties like steering_type, engine_type, proprietery fuel saving technology etc. But at its core, it's simply a car that has all the general attributes of a car like the number of wheels, transmission technique, engine type etc. So we can inherit an I20Asta object from a generic Car object.

```rb
class Car
  def available_fuel_options
    %w(petrol diesel lpg)
  end

  def steering_type
    'manual'
  end
end

class I20Asta < Car
  attr_accessor :owner
  def initialize(owner)
    @owner = owner
  end

  def steering_type
    'power'
  end
end
```

With separate objects for both a car and an i20asta car, we can test the 3 inheritance ideas described above.

```rb
a_car = Car.new
john_car = I20Asta.new('John')

# 1. subtype should be instance of supertype
john_car.is_a? I20Asta # true
john_car.is_a? Car # true

# 2. subtype should inherit properties from supertype
john_car.available_fuel_options
# ['pertrol', 'diesel', 'lpg']
# Note that the I20Asta class doesn't define or override the available_fuel_options method.

# 3. subtype should be able to override properties defined in supertype
a_car.steering_type # manual
john_car.steering_type # power
```

With a simple inheritance syntax, we are able to achieve all of these ideas in Ruby. Now let's see what it takes to achieve the same in Javascript.

### Javascript's Inheritance

Let's first create the constructor functions for both Car and I20Asta. Objects will be created only from these constructors.

```js
function Car(){
  this.wheels = ['front', 'back'];
}
Car.prototype.available_fuel_options = function(){
return ['petrol', 'diesel', 'lpg']
};
Car.prototype.steering_type = function(){
return 'manual';
};

function I20Asta(owner){
  this.owner = owner;
}
I20Asta.prototype.steering_type = function(){
  return 'power';
};
```

(Instead of adding properties to the constructor functions directly, we've added them in the function's prototype object. This way, the properties are shared by all of the objects created from these functions instead of occupying separate space in memory.) Note that we haven't yet implemented inheritance yet. There will be no association of any kind between objects created from these functions.

```js
var a_car = new Car();
var john_car = new I20Asta('John');

console.log(john_car instanceof I20Asta); // true
console.log(john_car instanceof Car); // false. Inheritance not yet implemented.
```

#### An aside about Prototype Object

When we ask a Javascript object for a property's value, it first looks for the property's presence right within the object. If it is present, then its value will be returned. If it's not present there, then Javascript will persist and ask the object's constructor function's prototype object for the value of that property. Only if it's not present even there, javascript will admit failure. Actually that's not true. If that object also has a reference to yet another prototype object, then javascript fill follow the trail upwards till it gets the value or till it reaches a dead-end. With this idea in mind, we can now make the john_car object inherit properties from the Car constructor by manipulating its prototype object reference. By default, the john_car object will have a reference to its constructor's prototype through its __proto__ property. Only because of that, the 'instanceof' check above passed.

```js
john_car.__proto__ === I20Asta.prototype // true
```

So far, our I20Asta function's prototype has nothing but a constructor property and the 'steering_type' property we added to it. It's of no use to us now considering that we need inheritance. To be able to inherit, what if we scrub I20Asta's current prototype object and make it point to another object? In particular, the object we want to inherit from - the Car? Let's do that right away.

```js
// the key line that enables inheritance
I20Asta.prototype = new Car();
```

The magic is done. But wait, since we scrubbed the old prototype object, we've lost the steering_type method we added to it. We need to add it again.

```js
I20Asta.prototype.steering_type = function(){
  return 'power';
};
```

*   Now our john_car object has access to all of these: it's own properties
*   properties added in its constructor's prototype object
*   properties defined in it's supertype's prototype object

We can now test the 3 inheritance ideas with success.

```js
// Redefine the objects
var a_car = new Car();
var john_car = new I20Asta('John');

// 1. subtype should be instance of supertype
john_car instanceof I20Asta;  //  true
john_car instanceof Car;  //  true

// 2. subtype should inherit properties from supertype
john_car.available_fuel_options();  //  ['petrol', 'diesel', 'lpg']

// 3. subtype should be able to override properties defined in supertype
I20Asta.prototype.available_fuel_options = function(){
  return ['petrol', 'diesel', 'lpg', 'electric']
};
a_car.available_fuel_options();  //  ['petrol', 'diesel', 'lpg']
john_car.available_fuel_options();  ['petrol', 'diesel', 'lpg', 'electric']
```

This method of implementing inheritance is called **"Prototype Chaining"**.

### Disadvantage of Prototype Chaining

With inheritance by prototype chaining, you can't have individual reference type properties inherited from the supertype. It will be shared across all objects. (Javascript reference types are objects, arrays, and user-defined custom objects, as opposed to primitive values. Variables referring to these items don't hold individual memory, instead they just act as pointers to the actual location of the reference types.) Notice that in the Car function, we have a wheels property which is an array. An array in javascript is a reference type. With this inheritance setup, try to ask both john_car and joe_car (another instance of I20Asta) for this property.

```js
john_car = new I20Asta('John');
joe_car = new I20Asta('Joe');

john_car.wheels // ['front', 'back']
joe_car.wheels // ['front', 'back']
```

All seem fine. Or so it appears. Let's say john has added another wheel to his car's side. To reflect this, we add another item to his wheels property.

```js
john_car.wheels.push('side');
john_car.wheels // ["front", "back", "side"]
```

Now ask joe_car for its wheels.

```js
joe_car.wheels // ["front", "back", "side"]
```

Inadvertently, we have updated Joe's wheels too! This is wrong. Joe didn't ask for an enhancement. As said earlier, this affects only the reference type properties. But that's enough of a deterrent to start searching for other inheritance methods.

### Combination Inheritance Pattern = Prototype Chaining + Constructor Stealing

That's a mouthful. But this is the most popular inheritance pattern used in javascript. At its core, it uses prototype chaining, but steals the supertype's constructor within the subtype constructor to rectify the problem discussed above. To implement this in the above example, you'd do this:

```js
function Car(){
  this.wheels = ['front', 'back'];
}
Car.prototype.available_fuel_options = function(){
  return ['petrol', 'diesel', 'lpg']
};
Car.prototype.steering_type = function(){
  return 'manual';
};
function I20Asta(owner){
  // NOTE: THIS IS THE CRUCIAL STEP. Calling the supertype's constructor enables access to its properties individually for the objects.
  Car.call(this);
  this.owner = owner;
}

I20Asta.prototype.steering_type = function(){
  return 'power';
};

// the key line that enables inheritance
I20Asta.prototype = new Car();

var john_car = new I20Asta('John');
var joe_car = new I20Asta('Joe');
```

All of the 3 inheritance tests discussed above works here too. You can test it. Now ask for wheels and try to manipulate them.

```js
john_car.wheels // ["front", "back"]
joe_car.wheels // ["front", "back"]

// add a wheeel to john's car in the side
john_car.wheels.push('side')
john_car.wheels // ["front", "back", "side"]

// Joe's car's wheels remain unaffected by the above change! It works ma!
joe_car.wheels // ["front", "back"]

joe_car.wheels.push('top')  //  for whatever reason!
joe_car.wheels // ["front", "back", "top"]
john_car.wheels // ["front", "back", "side"]
```

  Victory! We can now see that using this pattern, we are able to achieve perfect inheritance in javascript. Now go play. The World is your Javascripty Oyster!
