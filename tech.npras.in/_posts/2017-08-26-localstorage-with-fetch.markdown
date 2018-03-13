---
layout: post
title: "Using localstorage with Fetch API"
excerpt: "Cache the apis locally for a faster development"
---

When your development depends on a response from a remote api, you can't let your stride slow down because the api is slow. You can save a copy of it locally, and then use that in further requests to speed up the development.

The [Fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) is [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) based. When you make a http request with fetch, it returns a promise that would resolve with the response later when the api call returns.

So you can use the [Promise.resolve](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve) shortcut to return a promise that will always resolve to the response that you need for local development.

You could use the [localStorage api](https://developer.mozilla.org/en/docs/Web/API/Window/localStorage) of the browser to save the response as a json object after the first call. For subsequent requests, use this saved response with `Promise.resolve` like so.

1) First make a real http call and save the response in localstorage:

```js
fetch(url, httpOptions).then(response => {
  localStorage['remoteResponse'] = JSON.stringify(response)
  // do other stuff, maybe
})
```

2) Then, for subsequent calls, first check if a saved response is present, and if so, use it instead of making another api call.

```js
if (localStorage.remoteResponse) {
  return Promise.resolve(JSON.parse(localStorage.remoteResponse))
} else {
  fetch(url, httpOptions).then(response => {
    localStorage['remoteResponse'] = JSON.stringify(response)
    // do other stuff, maybe
  })
}
```

Depending on the complexity of the task at hand, this technique could add minutes to hours to your life.

And that's today's share of health tips. TC.
