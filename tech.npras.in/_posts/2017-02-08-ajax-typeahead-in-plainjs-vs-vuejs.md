---
layout: post
title: "Ajax Type Ahead in PlainJS vs VueJS"
excerpt: Learn to build a simple google-search like functionality in both PlainJS and VueJS. Also used is ES6.
---

A while ago, a certain [Wes Bos](http://wesbos.com/) released a free [Javascript course](http://wesbos.com/javascript30/) where he builds 30 apps using plain javascript.

The 6th exercise was to implement a simple ajax type ahead. I followed the exercise and refactored the code using [module design pattern](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript), and used proper forms of event listeners.

I also implemented it using Vue JS.

### Here's the Plain JS version
(click 'Rerun' and also open in a new tab to see the demo clearly:

<p data-height="300" data-theme-id="5444" data-slug-hash="GrXyLG" data-default-tab="js,result" data-user="npras" data-embed-version="2" data-pen-title="JS30 - 06 - Ajax Type Ahead - PlainJS " class="codepen">See the Pen <a href="http://codepen.io/npras/pen/GrXyLG/">JS30 - 06 - Ajax Type Ahead - PlainJS </a> by Prasanna N (<a href="http://codepen.io/npras">@npras</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

<br/>

### And here's the Vue JS version:
<p data-height="300" data-theme-id="5444" data-slug-hash="YNjjRG" data-default-tab="js,result" data-user="npras" data-embed-version="2" data-pen-title="JS30 - 06 - Ajax Type Ahead - VueJS" class="codepen">See the Pen <a href="http://codepen.io/npras/pen/YNjjRG/">JS30 - 06 - Ajax Type Ahead - VueJS</a> by Prasanna N (<a href="http://codepen.io/npras">@npras</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

<br/>

## Things I learned from this exercise

* ES6 is so cool. I'm almost done with [this book](http://amzn.to/2kMJh5E) covering all the features of ES6 (aka EcmaScript2015). It feels great to relate to and be able to use all the new cool features. Javascript is becoming powerful! Features that I used here:
  - fat arrow functions
  - array destructuring
  - const and let declarations
  - string interpolation 
  - concise method syntax to define a method in an object

* CoffeeScript might no more be needed! The fat arrow idea, the string interpolation, the destructuring etc were all pioneered by CS when it came up. But now that ES6 took the lessons and implemented them fairly, we won't be needing it anymore.

* I didn't use any library APIs to deal with the DOM. For sending ajax request, I was planning Wes would use the ever present (and only available) [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest). But he introduced me to the awesome [__fetch api__](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) which is a native api soon to be standardised (but already working in firefox and chrome). I also found and used the `'DOMContentLoaded'` event to implement the native alternative for jquery's famous `$(document.ready())`.

* The vast different between these 2 implementations of the same idea. Vue (and other frameworks) bring in a lot of abstractions and introduce new ways to think about the web interactions.

* The current Vue JS version is the final one I arrived at. But previously I used watchers and data function (in the component) to get the same result. I'd like to know the ideal way though.
