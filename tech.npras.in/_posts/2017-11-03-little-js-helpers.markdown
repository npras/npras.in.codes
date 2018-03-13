---
layout: post
title: "Little Javascript Helper Functions"
excerpt: "Things that end up in your 'services' or 'utilities' folder"
---

While working on a client's React app, I had to write several of these tiny functions to help my components with.

They range from url manipulation, js object manipulation, string/array changes, and input field formatters and normalizers.

Using several features of ES6/7 to write these was a joy.

---

Constructing a new url param object based on the current url. My use case here was: to extract all the params that are scoped and create a new urlparam without the scope.

```js
function extractUrlSearchParams() {
  let currentParams = new URLSearchParams(window.location.search)
  let filteredParams = new URLSearchParams()
  let md
  for (let param of currentParams) {
    md = param[0].match(/\[(.*?)\]/)
    if (md) { filteredParams.set(md[1], param[1]) }
  }
  const params = {
    param1: filteredParams.get('param1'),
    param2: filteredParams.get('param2'),
  }
  return params
}
```

---

Given an object, and a key, get the corresponding value out of the object. If there's no value, return a particular default value, and if needed return the value prefixed or suffixed with specific text.

```js
function get(object, property, {prefix='', suffix='', defaultVal='nope'} = {}) {
  if (object) {
    return object[property] ? `${prefix}${object[property]}${suffix}` : defaultVal
  } else {
    return defaultVal
  }
}
```

---

Check if 2 scalar arrays are equal. Array's a and b should be sorted beforehand.

```js
isEqualVals = (a, b) => (a.length == b.length) && (a.every((elem, i) => elem === b[i]))
```

---

Check if 2 different objects have same keys and same corresponding values.

```js
function isEqualObjects(a, b) {
  var aProps = Object.getOwnPropertyNames(a)
  var bProps = Object.getOwnPropertyNames(b)
  if (aProps.length != bProps.length) return false
  for (var i = 0; i < aProps.length; i++) {
    var propName = aProps[i]
    if (a[propName] !== b[propName]) return false
  }
  return true
}
```

---

Remove keys that have `null` value in them in an object.


```js
removeBlank(obj) {
  Object.keys(obj).forEach(key => (obj[key] == null) && delete obj[key])
  return obj
}
```
