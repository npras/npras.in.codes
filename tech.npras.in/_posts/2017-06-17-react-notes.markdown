---
layout: post
title: "ReactJS Notes"
excerpt: "Lessons learnt while learning from the fullstack-react book"
---

Recently I had to quickly ramp up on ReactJS for one of the client projects. I had less than a month to get started with it. My employer [Zanec](http://www.zanec.com/) gratefully bought this pdf ebook called as [Fullstack React](https://www.fullstackreact.com/). It's over 800 pages. I'm through with most of it. Topics like GraphQL, testing, deployment, relay etc are pending. These notes were taken when going through the rest of the book which covered react and redux in great detail.

The notes...

---

Why set state indirectly using `this.setState()`?
This method, in addition to setting the state object with new values, also is responsible for re-rendering the updated components in DOM.
That's why you should treat the state as an immutable object. You can mutate it only through setState.
Don't do this:

```js
const nextNums = this.state.nums.push(4); 
this.setState({ nums: nextNums });
```

This looks like we are using setState only. But unintentionally we are also modifying the state directly in 1st line. push modifies original array! Solution is to use concat if need be.

---

Need to figure out the meaning of this: TODO

`this.handleUpvote = this.handleUpvote.bind(this);` for custom component methods.

And using property initializers, this is not required it seems, if we use arrow functions for the custom component method.

---

__props vs state?__

- a parent component's state will go as props to a child component.
- props are passed to child component as its attributes
- the props are immutable by child. If a prop has to be changed, only the parent can do it via its state. The child can expose an event (eg: onNameChanged) as another attribute, to which the parent can pass a callback (handleNameChange) which is a custom component method in parent that can alter the state. (this is functional programming!)

from parent to child: data will flow down. from child to parent: event(s) will be propagating up.

---

__Handy framework for developing a React app from scratch__

1. Break the app into components
2. Build a static version of the app
3. Determine what should be stateful
4. Determine in which component each piece of state should live 
5. Hard-code initial states
6. Add inverse data flow
7. Add Server Communication

---

Forms are stateful. Any component that has a form will have to have its own state. And the input fields should have corresponding values in this state. And the `value` of each input should be tied directly to this state so that when state changes, the value changes too.

---

To force re-render a component, call `this.forceUpdate()` within the component.

---

Don't move all state management exclusively to server. Instead, do SM in 2 places - in server (db), and in react app's `this.state`. This is because this gives instant feedback to user when he updates anything. We don't have to wait for the server to get latest updated state.

---

DOM elems are represented as objects of type `ReactElement` in react js. So to render actual DOM elems, we first need to represent them as ReactElement objects, and then use `ReactDOM.render` to convert the js object to actual DOM elem.

```js
var boldElement = React.createElement('b', null, "Text (as a string)");
var mountElement = document.querySelector('#root');// Render the boldElement in the DOM tree
ReactDOM.render(boldElement, mountElement);
```

---

ReactElement vs ReactComponent. 2 different things.

---


static properties defined in a react component class:

```js
  static propTypes = {
    users: PropTypes.array.isRequired,
    initialActiveChatIdx: PropTypes.number,
    messages: PropTypes.array.isRequired,
  };
```

```js
// defines default values for props
  static defaultProps = {
    initialActiveChatIdx: 0,
  };
```

---

__context__ - experimental - when u want to pass down a prop from a parent to most of its children (and children of children), then it is an overhead to manually pass it from one to other. Using context, we can directly give access to child components certain props from parent, at any level.

2 props are needed to be defined in __parent__:
- childContextTypes - a static property
- getChildContext - a function that returns an object having the props exposed via context

Define this in the __child__ to retrieve the context:
- contextTypes

Then in the child, we can access the props in this object: `this.context.users` etc

---


prefer stateless components. if u _have to_ put state, then only do so if:
- it cannot be fetched from outside
- cannot be passed into the component as a prop
- it need not be sync'd across the app (eg: toggled buttons)

---

__props vs state => functional vs OO programming!__

If a component is seen as a 'pure function', then props are the args passed to the function. Passing the same props to the same function will always result in the same (predictable) output. The output is a function of just the props. So it's easy to reason about.

If a component is seen as an 'object', then it can hold data implicitly via 'instance vars'. So calling a function on an object that takes args (props) won't always result in same output all the time. It will depend upon the instance vars too. The output is a function of both props and 'instance vars'.

Avoid state as much as possible. But can't avoid it altogether! real world has state.

---

stateful components are created using es6 classes. stateless components are always created as functions that take props as arg. These functions are performant. But they do not have access to `this`, or render a `ReactElement` or create a new `ReactComponent`. Can't use refs and can't access dom too.

```js
const Header = function(props) { 
  return (<h1>{props.headerText}</h1>)
}
```

---

`this.props.children` gives access to all child components of a parent. We can use this when we don't know what children will be rendered ahead of time. Eg: sidebar navigation.

Newspaper component:

```jsx
<Container>
  <Article>this is article1</Article>
  <Article>this is article2</Article>
</Container>
```

Container Component:

```jsx
<div class='container'>
  {this.props.children}
</div>
```

Note that we could avoid `this.props.children` by exposing a prop to container:

```jsx
<Container articles={arrOfArticles}/>
```
---

use `props.children` to `compose` a component of multiple components. Do not prefer inheritance.

---

__"children are an opaque data structure"__ - It means: props.children can be any type, such as an array, a function, an object, etc. Since you can pass anything, you can never know for sure. So try to always use `React.Children.x` to handle children. x can be map or forEach, count, toArray etc

---

`React.CloneElement(child, {prop1: 'updaetd'})`. Props are immutable. So modifying children elems is not possible. So we have to use clone element to create a new copy of the child with updated property.

---

Clickhandlers are called with a `SyntheticMouseEvent` event object. It's a cross-browser wrapper.

---

__uncontrolled vs controlled components__

- uncontrolled: accessing a text input's value using ref and setting its value (DOM manipulation) using refs
- controlled: tying a state to the text input. this way, react will take care of DOM changes when this state changes. Prefer this always.

---

forms always get extended. maintaining state for each form input will get tedious. So maintain all form input in a single state item. It will be an object. Eg: state.fields.  Others can be accessed like state.fields.name, state.fields.email etc.

---

__redux__. redux works like this:

- state is managed by a store
- the view dispatches actions to the store
- the store receives the action which is just an object that has at least a `type` property. Eg: an increment action object will look like this: `{type: 'INCREMENT'}`. It can also have additional properties that are passed as arguments required for the action to the reducer. Eg: for a counter store, `{type: 'INCREMENT', amount: 2}`
- the store updates state by using a 'reducer' function. It takes 2 args: action object and current state. Combining these 2, the reducer function returns a new state
- the store notifies the view of the new state change. the view then rerenders

```js
function reducer(state, action) {
  if (action.type === 'INCREMENT') {
    return state + 1;
  } else {
    return state;
  }
}
```

A simple store using the above reducer function:

```js
function createStore(reducer) {
  let state = 0;   // NOTE: this is private. Only getState and dispatch can access it.

  const getState = () => (state);
  const dispatch = (action) => {
    state = reducer(state, action);
  };

  return {
    getState,
    dispatch
  };
}
```

---

reducer function should be a "pure function". It should NOT modify any of its surrounding world. Eg: it shouldn't mutate the state passed to it. It should return a new state object only.

---

reducer composition: break reducers into smallareducers, each managing a part of the state.

```js
function reducer(state = {}, action) {
  return {
    activeThreadId: activeThreadIdReducer(state.activeThreadId, action),
    threads: threadsReducer(state.threads, action)
  };
}

// if using react, then we can use combineReducers to get the same function as above
const reducer = combineReducers({
  activeThreadId: activeThreadIdReducer,
  threads: threadsReducer
});
```

---

2 types of components: __presentational and container__.

Container components will only talk with redux store and dispatch actions and specifies which presentational component to render.
Presentational components will just render html and won't deal with redux. Use js functions to build presentational components (which are also stateless).

---

The container component is the connector of the store to the presentational component.

---

Component design for a "Chat Intermediate" app. Each conversation with another person is a 'thread'.

- The state object looks like this:

```js
{
  activeThreadId: 'thread2',
  threads: [
    {
      id: 'thread1',
      title: 'Goku',
      messages: [
        {id: 'msg1', text: 'hi', timestamp: 2345555},
        {id: 'msg2', text: 'hw are u', timestamp: 2345555},
      ]
    },
    {
      id: 'thread2',
      title: 'Vegeta',
      messages: [
        {id: 'msg1', text: 'hi', timestamp: 2345555},
        {id: 'msg2', text: 'hw are u', timestamp: 2345555},
      ]
    },
  ]
}
```

- App - top level C. It's presentational. Just renders 2 container Cs. ThreadTabs and ThreadDisplay.

- ThreadTabs - container C. It manages active tab related state. It renders a presentational C 'Tabs' to which an onClick handler is passed to decide what to do when a tab is clicked. When it's clicked, this C will update activeThreadId in state

- ThreadDisplay - container C. It renders 2 presentational Cs: MessageList and TextInputSubmit. All required data and action handlers are passed down as props.

---

What container components do?

- map state to props
- map action dispatches to events

To do this, react-redux provides the `connect` function. It returns a container component if we pass it 3 functions: `connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`.

Here's how it works:

1. Call mapStateToThreadProps() with state
2. Call mapDispatchToThreadProps() with dispatch
3. Call mergeThreadProps() with the results of the two previous map functions (stateProps anddispatchProps)
4. Use the object returned by mergeThreadProps() to set the props on Thread

So now, the container Cs can be defined using connect like this:

```js
const ThreadDisplay = connect(
  mapStateToThreadProps,
  mapDispatchToThreadProps,
  mergeThreadProps
)(Thread);
```

where each of those functions will look like:

```js
const mapStateToThreadProps = (state) => (
  {
    thread: state.threads.find(t => t.id === state.activeThreadId)
  }
);

const mapDispatchToThreadProps = (dispatch) => (
  {
    onMessageClick: (id) => (
      dispatch({
        type: 'DELETE_MESSAGE',
        id
      })
    ),
    dispatch
  }
);

const mergeThreadProps = (stateProps, dispatchProps) => (
  {
    ...stateProps,
    ...dispatchProps,
    onMessageSubmit: (text) => (
      dispatchProps.dispatch({
        type: 'ADD_MESSAGE',
        text: text,
        threadId: stateProps.thread.id
      })
    )
  }
)
```

---

routing involves two primary pieces of functionality: (1) Modifying the location of the app (the URL) and (2) determining what React components to render at a given location.

For modifying the location of an app, we use links and redirects. In React Router, links and redirects are managed by two React components, Link and Redirect.For determining what to render at a given location, we also use two React Router components, Route and Switch.


react-router has Router, Route, Link and Switch components. They are used like this:

```js
const App = () => (
  <Router>
    <Link to='/atlantic'>
      <code>/atlantic</code>
    </Link>

    <Route path='/atlantic' component={Atlantic} />
    <Route path='/pacific' component={Pacific} />
  </Router>
);
```

---

Router is the container component that gives the child routing components (Link, Route, Redirect etc) access to location and history object via context. It's defined like this:

```js
class Router extends React.Component {

  static childContextTypes = {
    history: React.PropTypes.object,
    location: React.PropTypes.object,
  };

  constructor(props) {
    super(props);

    this.history = createHistory();
    this.history.listen(() => this.forceUpdate());
  }

  getChildContext() {
    return {
      history: this.history,
      location: window.location,
    };
  }

  render() {
    return this.props.children;
  }
}
```

Note that when the url (history) changes, it is Router's responsibility to re-render. This will also re-render the Routes within thereby they'll do the correct work.

---

Route renders the given component only if the current url matches the path.

```js
const Route = ({ path, component }, { location }) => {
  const pathname = location.pathname;
  if (pathname.match(path)) {
    return (
      React.createElement(component)
    );
  } else {
    return null;
  }
};

Route.contextTypes = {
  location: React.PropTypes.object,
};
```

---

Link is responsible for pushing new url to history.

```js
const Link = ({ to, children }, { history }) => (
  <a
    onClick={(e) => {
      e.preventDefault();
      history.push(to);
    }}
    href={to}
  >
    {children}
  </a>
);

Link.contextTypes = {
  history: React.PropTypes.object,
};
```

---

Redirect looks like this:

```js
class Redirect extends React.Component {

  static contextTypes = {
    history: React.PropTypes.object,
  }

  componentDidMount() {
    const history = this.context.history;
    const to = this.props.to;
    history.push(to);
  }

  render() {
    return null;
  }
}
```

So whole workflow is:

* When a Link is clicked, the url is changed using history api.
* Since the Router subscribes to changes for history object, it gets notified of the url change. It then re-renders.
* The routes present within also re-render - either showing or not-showing their component based on the current path.
