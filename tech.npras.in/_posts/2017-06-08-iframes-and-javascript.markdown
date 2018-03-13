---
layout: post
title: "Javascript and iFrames"
excerpt: "Dynamically create iFrames using JS and use iFrame Resizer library"
---

Consider this web app requirement:

* You have a form in a page that submits the values to the server via a GET request.
* After submitting the form, the user is taken to another page. Since it was submitted via GET, you can see the submitted values in the current page's url as query strings
* The requirement now is to load an iframe into this page whose source url, via the src attribute, will have the same query strings that the current page has attached to it. We can use [UrlSearchParams](https://github.com/jerrybendy/url-search-params-polyfill) to extract the query strings to construct the iframe's url
* Here's another requirement: the loaded content within the iframe will have to be displayed fully. The iframe should expand to "contain" all of the content without using any scrollbars. The challenge here is that the parent page won't be knowing the height and width of the loaded page beforehand. So it can't dynamically adjust the iframe's size.
* Enter [iFrame Resizer](https://github.com/davidjbradshaw/iframe-resizer). This library provides 2 js files. One is to be included in the parent page hosting the iframe, and the other is to be included in the page sourced by the iframe.

Here's how the hosting page might look like:

```html
<html>
<head>
  <style>
    iframe{
      width:100%;
    }
  </style>

  <script src="/js/urlsearchparams_polyfill.js"></script>
  <script src="/js/iframeResizer.min.js"></script>

  <script>
  document.addEventListener('DOMContentLoaded', function(){
    var currentParams = new URLSearchParams(window.location.search);
    var iframe = document.createElement('iframe');
    iframe.src = 'http://iframesourcepage.com/path?' + currentParams.toString();
    document.body.appendChild(iframe);
    var iframes = window.iFrameResize({
      log: false,
      autoResize: true,
      heightCalculationMethod: 'max',
    });
  });
  </script>
</head>


<body>
</body>
</html>
```

What this does is:
* on page load, it extracts the query strings from url using the urlsearchparams polyfill
* an iframe element is dynamically created with its url appended with these query strings. The created iframe is attached to the current page's html document
* the iframe resizer is initialized with options that allow it to expand/shrink automatically based on its content's size.

For this to work as intended, the hosted page within the iframe must have the 2nd js from the resizer lib. Add this to the `<head>` of that page:

```html
<script src="/assets/iframeResizer.contentWindow.min.js"></script>
```

Now things will work as expected.
