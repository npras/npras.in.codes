---
layout: post
title: A Primer about Javascript's Prototype
categories: code
excerpt: Note to self - No more googling to understand prototypes
---

If you are just learning Javascript, understanding clearly the concept of a javascript prototype might be crucial, but at the same time impossible. All the popular google search results point you to well explained articles that come from expert mouths. As in the below links.

*   [http://yehudakatz.com/2011/08/12/understanding-prototypes-in-javascript/](http://yehudakatz.com/2011/08/12/understanding-prototypes-in-javascript/)
*   [http://javascriptissexy.com/oop-in-javascript-what-you-need-to-know/](http://javascriptissexy.com/oop-in-javascript-what-you-need-to-know/)
*   [http://javascriptissexy.com/javascript-prototype-in-plain-detailed-language/](http://javascriptissexy.com/javascript-prototype-in-plain-detailed-language/)
*   [http://javascriptissexy.com/javascript-objects-in-detail/](http://javascriptissexy.com/javascript-objects-in-detail/)
*   [https://developer.mozilla.org/en/docs/Web/JavaScript/Inheritance_and_the_prototype_chain](https://developer.mozilla.org/en/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
*   [http://blog.pluralsight.com/understanding-javascript-prototypes](http://blog.pluralsight.com/understanding-javascript-prototypes)

But I find them lacking simple explanation. While you can take the following explanation as the Final Word of God About Prototype, I would treat this as a "explain to me like I'm 5" guide and then proceed to checkout other info sources to cross-check and deepen the memory groove you I'd've formed. So, what's a prototype in javascript? When a function is created, it gets a **special** property called 'prototype' which is just an Object. It's special because javascript specifically looks for a property named 'prototype' in a function or an object when it has to do certain operations. For the time being, I'd advise you to see 'prototype' as just a word, and not as a word with a meaning associated to it. Forget the association of the dictionary meaning of prototype to the word prototype. See it as just another property on a js object.  

### A Function's Prototype

So again, when a function is created, it gets a special property called 'prototype' which is just an Object.  

```js
function sayHi(){
return 'hi';
}
typeof sayHi.prototype; // 'object'
```

You can check all the examples in this post by opening Chrome's javascript console or by typing "node" if you have installed NodeJS installed. Javascript objects have their own properties. So if a function's prototype is an object, then it too must have some properties right? Let's find out. Using the handy, in-built `Object.getOwnPropertyNames(object)` function, we can get an array of all the own properties of an object. Let's find out the own properties of the prototype object of sayHi function.  

```js
Object.getOwnPropertyNames(sayHi.prototype)
// ["constructor"]
```

The prototype object of sayHi function has a single property called as 'constructor'. Well, what is it? Let's find out.  

```js
sayHi.prototype.constructor
// outputs:
// function sayHi(){
// return 'hi';
// }
```

  Hmm. Is it the same thing as our sayHi function? Let's find that too.

```js
sayHi.prototype.constructor === sayHi
// true
```

Yes indeed. A function's prototype object has, initially, a special property called as 'constructor' which simply points back to the original function. Circle of life.

### The purpose of the prototype property (or Why it is named prototype)

When you create an object from a function, and when you ask the object the value of any property, javascript first looks for the property right within the object itself. If it finds, the property's value is returned. If not, then instead of simply stopping, javascript will then ask the function's prototype for the property's value. Consider this:

```js
function Person(name, age){
  this.name = name;
  this.age = age;
  this.bio = function(){
    return "I'm " + this.name + ", and my age is " + this.age + ".";
  };
}
```

```js
var john = new Person('John', 23);
john.name // 'John'
john.age // 23
john.bio() // I'm John, and my age is 23.
```

When we ask the john object for a 'name' property, it returns 'John' because it has a property named 'name' of its own. Similarly we can ask it's age and also ask it to speak its bio.

```js
Object.getOwnPropertyNames(john);
// ["name", "age", "bio"]
```

Now let's add some additional properties, but not to the john object, or the Person function, but to the Person function's prototype object.

```js
Person.prototype.species = "Sapiens"
Person.prototype.sayName = function(){
  return "Hi, I'm" + this.name;
};
```

  We can even check that these 2 properties are now newly added only to the prototype object, and not to any of the objects created via Person (eg: john object).  

```js
Object.getOwnPropertyNames(Person.prototype);
// outputs:
// ['constructor', 'species', 'sayName']

Object.getOwnPropertyNames(john);
// still the same as before
// ["name", "age", "bio"]
```

Now try asking the john object for a 'species' property, and also lets ask it to say its name, both of which are not own props of john.  

```js
john.species
// 'Sapiens'

john.sayName()
// "Hi, I'm John"
```

As you can see, the john object now has simply become awesome. When we ask it about a property that's not its own, it still looks up and asks its constructor's prototype object to see if it has these properties and returns their values if they exist. Did you also notice that the john object is able to do this even when the species and sayName props are added to prototype, *after* john object's creation? That's too hot an awesomeness to handle. Now let's do one more thing. From the Person function, let's create another object named 'Joe'.  

```js
var joe = new Person('Joe', 35);
joe.name // 'Joe'
joe.age // 35
```

  Let's ask Joe too his species and ask him to speak his name.

```js
joe.species // 'Sapiens'
joe.sayName() // "Hi, I'm Joe"
```

  From this we can infer that the properties assigned to the prototype object is accessible to all of the objects created from the function. But why? What difference does it make to have these properties defined right within the function itself? Functions and objects are real things occupying real space in the computer's memory. So when we define a property in the function, and an object is created, memory is allocated for those properties in the object. Even though john and joe both have same own properties - name, age, bio - because they are different objects, these properties too tend to take up separate space. On the other hand, the properties defined on the prototype - species and sayName - they are not duplicated. At any given instance in the application's runtime, these props occupy only one memory space. So when we ask both john and joe for these properties, javascript just gets the value from this shared location for both of them.

### An Object's Prototype

So far we saw about a function's prototype property and its uses. Does an object have a prototype property too? It kinda has. An object's prototype simply points to the object's constructor's prototype. (To refresh, an object's constructor is the function that was used to create the object. john and joe's constructor are both the Person function.) But it's not accessible as the ecmascript implementation prohibits it. But still browsers and nodejs implementations expose an object's prototype via the __proto__ property.

```js
john.__proto__ === john.constructor.prototype
// true
```

We can also check this in 2 more ways:

```js
Object.getPrototypeOf(john) === Person.prototype
// true

Person.prototype.isPrototypeOf(john)
// true
```

### Conclusion

A Prototype is neither a mystical Unicorn nor "version 0" of anything (the dictionary connotation). It's just a property on a javascript function, that initially has a 'constructor' property. The constructor property of the prototype object is just a pointer to the function itself. Properties defined in a prototype object are accessible to all the objects created via the function. An object doesn't have a prototype of its own. But its internal prototype property is simply a pointer to the object's constructor's prototype.
