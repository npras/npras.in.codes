---
layout: post
title: "Understanding Javascript Promises"
excerpt: "Trying out examples from the internet to get to know Promises better"
---

The new `fetch API` to make ajax requests is promise based. It looks like this:

```js
fetch(url)
.then(data => console.log(data))
.catch(error => console.log(error))
```

The old `XMLHttpRequest` way of sending ajax requests is not promise based. But we can wrap it in a function to make it so:

The wrapper api we'd design might be:

```js
my_fetch(url)
.then(data => console.log(data))
.catch(error => console.log(error))
```

This is implemented like so:

```js
function my_fetch(url){
  executorFn = (resolve, reject) => {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = () => {
      if (xhr.status == 200) {
        resolve(xhr.response);
      } else {
        reject(Error(`On non 200: ${req.statusText}`));
      }
    };

    xhr.onerror = (e) => {
      reject(`On Error: ${Error(e)}`);
    };

    xhr.send(null);
  };

  return new Promise(executorFn);
}
```

You can try this out in chrome by visiting Github.com, and then opening the console in devtools and defining the `my_fetch` function there, and then calling it like so:

```js
get('https://github.com/npras.keys')
.then(function(data){
  console.log(`Got Data! ${data}`);
})
.catch(function(error){
  console.log(`Failed: ${error}`);
})
```

After a very small network delay, you'll see my public keys in the console.


When a promise is created using the `new Promise(executorFn)` constructor, the created promise is always in the "pending" state. You can verify it like so:

```js
np = new Promise((resolve, reject) => {})
// np will be:
// Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
```

But if you already know, and have what you want from the promise call, you can return that value directly right? No. To keep consistent with the return values, you need to return a promise that is __already resolved__.

There's then the `Promise.resolve()` and `Promise.reject()` that does these jobs.

```js
resolvedPromise = Promise.resolve(4);
// Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: 4}
```

You can then proceed to access the value using `then`.


To try this out, we'll create a cached version of our `my_fetch` function. This will take an url and fetch its data using our original `my_fetch` function. Then, when we call the `my_cached_fetch` again with the same url, no network calls will be made. The data will be 'resolved' quickly from the cache.


Here's the function:

```js
const cache = {};
function my_cached_fetch(url){
  if (cache[url]) {
    console.log('already cached...');
    return Promise.resolve(cache[url]);
  }

  return my_fetch(url)
    .then((result) => {
      console.log('fetching and caching for the first time...');
      cache[url] = result;
      return result;
    })
    .catch((error) => {
      throw Error(`Can't find url: ${error}`);
    });
}
```

And it is called like so:

```js
my_cached_fetch('https://github.com/npras.keys')
.then(function(data){
  console.log(`Got Data! ${data}`);
})
.catch(function(error){
  console.log(`Failed: ${error}`);
})
```

Call it again and see the difference in the statements that are 'console.log'ed.
