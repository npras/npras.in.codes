---
layout: post
title: "VueJS: Implementing Proxy behaviour of Data object"
excerpt: "This isn't about Vue. It's about Javascript, and how it was used to design an interesting behaviour."
---

The [official Vue guide](https://vuejs.org/v2/guide/instance.html#Properties-and-Methods) for the Vue instance tells that the instance object proxies all properties found in the instance's `data` object.

```js
var data = { a: 1 }

var vm = new Vue({ data })

vm.a === data.a // -> true
// setting the property also affects original data
vm.a = 2
data.a // -> 2

// ... and vice-versa
data.a = 3
vm.a // -> 3
```

We'll replicate this behaviour in a vanilla JS function. You don't have to know about VueJS to follow the rest of the post. It's just about javascript.

```js
function myVue(opts){
  this._data = opts.data;
  Object
    .keys(opts.data)
    .forEach(key => {
      props = {
        configurable: true,
        enumerable: true,
        get() { return this._data[key]; },
        set(val) { this._data[key] = val; }
      }
      Object.defineProperty(this, key, props);
    });
}
```

Let's try this `"myVue"` function with the same example as from the official guide.

```js
let data = { a: 1 };

let vm = new myVue({ data });

console.log(vm.a === data.a);

// setting the property also affects original data
vm.a = 2;
data.a // -> 2

// ... and vice-versa
data.a = 3;
vm.a // -> 3
```

It works!

### But how?

The key JS feature that enables this behaviour is [Getters and Setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Defining_getters_and_setters) of the javascript object.

It lets you define properties which are not hard-coded, instead are represented by functions that dynamically return values.

### Conclusion
I came to know about this technique after finding it in the [VueJS source code](https://github.com/vuejs/vue). The magic happens in the [src/core/instance/state.js](https://github.com/vuejs/vue/blob/dev/src/core/instance/state.js) file.

The `initData` function does the work. It loops over `data`'s keys and asks the `proxy` function to define the proxied properties on the Vue instance itself.
