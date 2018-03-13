---
layout: post
title: "ReactJS: Using Formatters and Normalizers in ReduxForm"
excerpt: "Separation of concern applied to html text inputs"
---

The greatest invention since the discovery of html input text box is the redux form concept of formatters and normalizers to the input fields.

We always wanted to transform what the user entered into something else before putting the value in the database. And we always wanted to restrict the output that the user sees when he enters input. These 2 are separate concerns that should've been separate from the beginning.

Here's an example application of the concept.

I have a text input that takes price in USD as its input. In the database, it is represented by a simple number data type. But suppose when the user starts typing I want a '$' sign to be prefixed to the values entered in the field, and I want the user to only type numbers in there. Any other characters should be restricted. And I want a comma to be placed automatically in the thousandth separator.

These 2 functions - formatter and normalizer funcs - will be executed on each input event.

The formatter does this:

* replace any non-digit chars with empty strings. Only digit chars are allowed
* Using [special regex](https://tech.npras.in/advanced-regex/) a comma is inserted as thousandth separator (one for each 3 digits)
* If the first char isn't '$', prefix the result with a '$'

```js
export function formatCurrency(val) {
  val = val.replace(/[^\d]/g, '');
  val = val.replace(/(\d)(?=(\d\d\d)+(?!\d))/g, "$1,");
  if (val[0] !== '$') {
    val = '$' + val;
  }
  return val
}
```


The normalizer does this:

* replace any non-digit chars with empty strings. Only digit chars are allowed as this is what needs to go into database.

```js
export function normalizeCurrency(val, prevVal) {
  return val.replace(/[^\d]/g, '')
}
```

And finally these 2 are passed to the input field component.

```js
<Field name="spend" component="input" type="text" format={formatCurrency} normalize={normalizeCurrency} className="input input-text" />
```
