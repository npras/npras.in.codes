---
layout: post
title: "ROT13 Implementation in Javascript and Ruby"
excerpt: "Wikipedia says  \"The algorithm provides virtually no cryptographic security, and is often cited as a canonical example of weak encryption\""
---

While rot13 cannot be used for encrypting important information, it can be used in places where security is not that much of a concern.
One typical ues case: Suppose we are building an amazon product review site. All of our affiliate links in the site are to be used carefully. If Amazon detects abuse (multiple illegal clicks), then they'll cancel the affiliation and we won't be earning any commission off the clicks.

One main concern is our links can be abused by internet bots. They'll be crawled and accessed without our permission. This would lead Amazon to suspect us and might lead to account cancellation. In that case, we can rot13 the affiliate links in the server (using say ruby), and when the page loads in the browser, javascript can be used to decrypt it back to the actual links using rot13 (rot13 is its own reverse).


So here's rot13 in ruby:

```ruby
def rot13(str)
  str.tr("A-Ma-mN-Zn-z","N-Zn-zA-Ma-m")
end
```


And here's rot13 in javascript:

```js
function rot13(str) {
  var func = function(c) {
    return String.fromCharCode(
      (c <= "Z" ? 90 : 122)
      >=
      (c = c.charCodeAt(0) + 13)
      ?
        c
      :
        c - 26
    )
  }
  return str.replace(/[a-zA-Z]/g, func)
}
```

Documentation on the methods used:

* ruby's [String#tr](https://ruby-doc.org/core-2.2.0/String.html#method-i-tr)
* javascript's [String.prototype.replace with function as 2nd argument](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_function_as_a_parameter)
* javascript's [String.prototype.charCodeAt](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)
* javascript's [String.fromCharCode](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode)
