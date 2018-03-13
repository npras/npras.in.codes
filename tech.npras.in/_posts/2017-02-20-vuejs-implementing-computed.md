---
layout: post
title: "VueJS: Implementing Computed properties"
excerpt: "Learn how the caching behaviour of computed properties is coded."
---

The [__Official Vue guide__](http://localhost:4000/v2/guide/computed.html#Computed-Caching-vs-Methods) for "Computed Properties" explains the caching functionality like this:

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

> computed properties are cached based on their dependencies. A computed property will only re-evaluate when some of its dependencies have changed. This means as long as message has not changed, multiple access to the reversedMessage computed property will immediately return the previously computed result without having to run the function again.

In this post, I'll explain how the caching part is implemented. (May be by next week I'll make a post about how the computation is done when the dependencies are changed.)

(Check [__this post__](/vuejs-proxy-data-object/) to see how the proxy behaviour of the data object is implemented.)

The basic idea is very simple:

```js
function age() {
  console.log('age...');
  return 42;
}

function Person(name) {
  this.name = name;
  this.age = age();
}

john = new John('John');

john.name // 'John'
john.age // 42
john.age // 42
```

This is a simple Person function with name and age property. Notice that the every time 'age' is accessed off an instance (john), the `consolg.log('age...')` __doesn't__ get executed. It's only executed at the very first time john is instantiated. The value returned by the `age()` function call is simply stored in memory.

In [VueJS source code](https://github.com/vuejs/vue), the computed property's caching behaviour is implemented using this idea.

As in the previous post, you need not know about VueJS to understand this post. This is just an exercise in reading and understanding an expert's code.

We'll be building off the `myVue` constructor function that we created in the last post that emulates the actual `Vue` function.

```js
function myVue(opts) {
  this._data = opts.data;
  Object
    .keys(opts.data)
    .forEach(key => {
      sharedPropDef.get = () => this._data[key];
      sharedPropDef.set = (val) => { this._data[key] = val; };
      Object.defineProperty(this, key, sharedPropDef);
    });

  if (opts.computed) initComputed(this, opts.computed);
}
```

The new addition is the last line. If `computed` prop is present in the passed options object, then we call the `initComputed` function.

```js
function initComputed(vm, computed) {
  vm._computedWatchers = Object.create(null);

  for (const key in computed) {
    value = computed[key];
    vm._computedWatchers[key] = new Watcher(vm, value);

    if (!(key in vm)) {
      sharedPropDef.set = noop;
      sharedPropDef.get = function () {
        const watcher = vm._computedWatchers && vm._computedWatchers[key];
        if (watcher) return watcher.value;
      };
      Object.defineProperty(vm, key, sharedPropDef);
    } // if
  } // for 
}
```

The `noop` and `sharedPropDef` referenced are declared at the top like so:

```js
const noop = function () {};
const sharedPropDef = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
};
```

Since we are calling the computed property directly on the Vue instance itself, we know that we'd need a property on the instance for each computed property. The last line does that with `Object.defineProperty`. We are defining each of those props as props with a dummy setter and a proper getter. ([getters and setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Defining_getters_and_setters)).

For each computed property, a `watcher instance` is created, and stored in an internal map object, inside the Vue instance: `_computedWatchers`. Later on, we lookup this map using the computed property's name to fetch its corresponding watcher instance.

The watcher instance is created using the `Watcher` class. It's just an ES6 glorified entity defined exactly as the `Person` constructor function shared earlier.

```js
class Watcher {

  constructor(vm, valueFn) {
    this.vm = vm;
    this.getter = valueFn;
    this.value = this.get(); // this is the key line
  }

  get() {
    let value;
    const vm = this.vm;
    value = this.getter.call(vm);
    return value;
  }

}
```

You can now see the cached computed property in action:

```js
data = { msg: 'hello' };

computed = {
  reversedMsg() {
    console.log('reversed...'); // printed only once during vm initialization
    return this.msg.split('').reverse().join('');
  },
  upperCased() {
    console.log('uppercased...'); // printed only once during vm initialization
    return this.msg.toUpperCase();
  }
};

opts = { data, computed };

vm = new myVue(opts);

console.log(vm.reversedMsg); // 'olleh'
console.log(vm.reversedMsg); // 'olleh'

console.log(vm.upperCased); // HELLO
console.log(vm.upperCased); // HELLO
```

The full working example is split as 2 files: [__vuejs-source.js__](https://gist.github.com/npras/c70c76e3eabb61a8f2b34a6fe585a6da) and [__watcher.js__](https://gist.github.com/npras/df1d3390a75c1d66185d5cf466808696).
