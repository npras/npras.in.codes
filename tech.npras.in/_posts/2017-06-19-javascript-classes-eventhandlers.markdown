---
layout: post
title: "Javascript Class Methods, Binding and Event Handlers"
excerpt: "Understanding `this.handleX = this.handleX.bind(this)`"
---

A typical React component with event handling (say `onclick`) would look like this:

```js
class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { value: 0 }
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick(e) {
    this.setState({
      value: (this.state.value + 1)
    })
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick}>Click me!</button>
      </div>
    )
  }
}
```

This line: `this.handleClick = this.handleClick.bind(this)` always intrigued me. It's just a javascript quirk, not mandated by react.

https://facebook.github.io/react/docs/handling-events.html helped explain things better.

So here's what's going on:

* when we mention `this.handleClick` within the jsx, it's obvious that we are not calling the function, but just passing the function as a whole, to be called _later_ when the actual event is fired.

* Within the `handleClick` function, we are referring the object instance using `this`. We do this because we need access to the `setState` method within `handleClick`.

* The catch is, when the event does fire, the `handleClick` will be invoked without any reference to `this`. It will be just passed as a function, and not as a method of the react object.

* The solution: `this.handleClick = this.handleClick.bind(this)` fixes this because it **binds** handleClick to the react object once and for all. So wherever the function is passed, it will always be run in the context of the react object. So things will work as expected.

Before knowing this, I was confused and had this question: If `handleClick` is not bound to `this`, then how did we get access to it _using_ `this`? Like this: `this.handleClick.bind(this)`. Now I understand. Within the context of the class, `handleClick` is bound. It's only for the sake of the event handler we do this dance.

Of course, we could avoid this using arrow functions for `handleClick` as it takes the encompassing function's `this` for itself.
