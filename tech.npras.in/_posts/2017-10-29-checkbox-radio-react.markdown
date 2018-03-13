---
layout: post
title: "ReactJS: Integrating Checkbox and Radio with React form"
excerpt: "Simple things, but a bit of a bump the first time"
---


First, define your own wrapper components.
```js
import React from 'react'
import { Field } from 'redux-form'

export default function Checkbox(props) {
  return (
    <div>
      <Field component="input" type="checkbox" name={props.name} id={props.name} />
      <label htmlFor={props.name}>
        <span>{props.label}</span>
      </label>
    </div>
  )
}
```

```js
import React from 'react'
import { Field } from 'redux-form'

export default function RadioButton(props) {
  return (
    <div>
      <label htmlFor={props.value}>
        <Field component="input" type="radio" name={props.name} id={props.value} value={props.value} />
        {props.label}
      </label>
    </div>
  )
}
```

Then, use them like so:

```js
<Checkbox name="genderMale" label="Male" />
<Checkbox name="genderFemal" label="Female" />


<RadioButton name="newsletterSubscription" label="Yes" value="yes" />
<RadioButton name="newsletterSubscription" label="No" value="no" />
```

Notice that, for `Checkbox`, the name is not the same for the 2 related checkboxes. Normally we'd gather all these checkbox inputs into an array variable of the form and send it to the server. But here, it's not the case. `redux-form` doesn't provide that facility as far as I can tell. Instead it allows us to track each of the checkbox item as a unique property of its own. Whoever is processing these inputs will have to decide how to treat the values of these individual items.
