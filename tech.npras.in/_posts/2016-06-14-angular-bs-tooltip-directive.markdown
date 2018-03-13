---
layout: post
title: Building Bootstrap Tooltip Directive in AngularJS
categories: code
excerpt: where I take a small step in advancing my AngularJS knowledge
---

I came across a good way to practise directives from Thinkster.io. They have a tutorial where they build a bootstrap tab directive with a nice api. Here it is: [https://thinkster.io/angular-tabs-directive.](https://thinkster.io/angular-tabs-directive.) In similar vein, here I build a bootstrap tooltip directive.

##### The API

Currently you'll need this html to get your tooltip 'bootstrappified':

```html
<button type="button" class="btn btn-default" data-toggle="tooltip" data-placement="left" title="Tooltip on left">Tooltip on left</button>
```

There are 3 different components related to tooltip here:

*   the title attribute that gives the tooltip text
*   data-placement attribute gives the direction
*   data-toggle attribute is required to trigger the corresponding bootstrap js function.

I'd like to simplify this. I propose this api:

```html
<button ui-title="left: Tooltip on left">Tooltip on left</button>
```

That's it. The direction specifier and the tooltip content, both go in one attribute. Here's the angular directive I built for this api:  

See the Pen [Angular BS Tooltip](http://codepen.io/npras/pen/EKXYEP/) by Prasanna N ([@npras](http://codepen.io/npras)) on [CodePen](http://codepen.io).
