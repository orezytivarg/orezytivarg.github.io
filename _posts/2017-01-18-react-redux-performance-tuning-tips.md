---
id: react-redux-performance-tuning-tips
title: React Redux 성능 튜닝 팁
category: Redux
---

[원문보기](https://medium.com/@arikmaor/react-redux-performance-tuning-tips-cef1a6c50759#.64yt91nsu)

# React Redux 성능 튜닝 팁

When writing a complex React app, you might find yourself struggling with rendering performance issues.
This article will give you an overview of the tools and techniques used to detect and fix this performance bottlenecks.

## The Problem is in React’s rendering process

When a component calls `this.setState()`, React re-renders the DOM in two stages:

    React’s internal Virtual DOM is re-rendered
    The diff between the previous virtual DOM and current DOM is calculated and changes are applied to the actual DOM

If the first step takes too long, re-rendering will be slow.

## Avoiding re-rendering of pure components

The basic principle in React rendering optimization is to let React know that it doesn’t need to render again because we know the resulting DOM will have no changes.
We can signal React not to render in a couple of ways:

### Returning the same element reference

If the render method returns the same ref, React will assume it was unchanged.

```js
class MyComponent extends Component {
  text = ""
  renderedElement = null

  _render() {
    return <div>{this.props.text}</div>
  }

  render() {
    if (!this.renderedElement || this.props.text !== this.text) {
      this.text = this.props.text
      this.renderedElement = _render()
    }
    return this.renderedElement
  }
}
```

You can also use the memoize function from lodash if you don’t want to implement the caching yourself.

```js
import memoize from 'lodash/memoize'

class MyComponent extends Component {
  _render = memoize((text) => <div>{text}</div>)

  render() {
    return _render(this.props.text)
  }
}
```

### Returning false from shouldComponentUpdate

React calls this method to check if it should re-render, the default implementation always returns true.
React provides a shallowCompare function (we can get it from other libraries as well) that checks the equality of the top level properties in two objects. Using this function, we can implement a pure component like this:

```js
import shallowCompare from 'react-addons-shallow-compare'

export default class PureComponent extends Component {
  shouldComponentUpdate(nextProps, nextState) {
    return shallowCompare(this, nextProps, nextState);
  }

  render () {...}
}
```

You can also write you own logic to deciding if to re-render or not, but make sure this function is very fast. If the function is slow, we’ll be back were we started, as it will be called every time React re-renders.

### High Order Components

We can use high order components to implement this optimizations and reuse them.
Actually, there is already a library called Recompose that includes lots of generic high order components for us to use.

## Redux and connect()

When using redux, we use the higher-order component connect(). This component retrieves the store for the context and calls mapStateToProps when the state changes. The connected component is re-rendered only when the relevant values in the current are actually changed (comparison is also done using shallowCompare). For example:

```js
// only re-renders when prop1 changes
connect(state => ({
    prop1: state.prop1
}))(SomeComponent)
```

There is a ‘gotcha’ here, if mapStateToProps function performs some calculation, it can cause connect() to re-render unnecessarily.

```js
// this is ok
connect(state => ({
    hasSomething: this.prop1 === 5 // true===true
}))(SomeComponent)

// this is NOT OK
// computed data is a different object every-time even if props were un-changed
connect(state => ({
    computedData: {
        height: state.height,
        width: state.width
    }
}))(SomeComponent)
```

### Fixing the issue with reselect

[reselect](https://github.com/reactjs/reselect) helps us fix this issue by helping us declare the dependencies to a derived state and handling the caching for us, similar to the way memoize does:

```js
import {createSelector} from 'reselect'
const selectComputedData = createSelector(
    state => state.height,
    state => state.width,
    (height, width) => ({
        height,
        width
    })
)
connect(state => ({
    computedData: selectComputedData(state)
}))(SomeComponent)
```

## Pure Render Anti-Pattern

When using a pure component, pay special attention to arrays and functions. Arrays and functions create new refs so it’s up to you to create them only once and not during every render.

### Functions

```js
// NEVER do this
render() {
  return <MyInput onChange={this.props.update.bind(this)} />;
}
// NEVER do this
render() {
  return <MyInput onChange={() => this.props.update()} />;
}
// Instead do this
onChange() {
    this.props.doUpdate()
}
render() {
  return <MyInput onChange={this.onChange}/>;
}
```

We can also avoid binding inside the component by binding in an higher-order component like recompose.withHandlers() or redux.connect()

```js
// recompose
@withHandlers({
  onChange: props => event => {
    props.update(event.target.value)
  }
})
class SomeComponent extends Component {
  render() {
    return <MyInput onChange={this.props.onChange}/>;
  }
}

// redux
@connect(null, (dispatch, ownProps) => {
  onChange: event => {
    dispatch(actions.updateValue(event.target.value))
  }
})
class SomeComponent extends Component {
  render() {
    return <MyInput onChange={this.props.onChange}/>;
  }
}
```

### Arrays

```js
// NEVER do this, if there are no items, SubComponent will render every time!
render() {
    return <SubComponent items={this.props.items || []}/>
}
// This will avoid re-rendering
const EMPTY_ARRAY = []
render() {
    return <SubComponent items={this.props.items || EMPTY_ARRAY}/>
}
```

## Debugging

Besides adding console.log() to render() methods, finding the cause for performance issues is done mainly with 2 tools.
This is a quick summary, but for more info and screen-shots read Benchling’s part-1 and part-2 blogs post about React performance engineering .

### Chrome DevTools Profiler

Choose the Timeline tab in Chrome DevTools, and record while you perform an action in your application. The timeline will show you how much time the browser spent executing code, and how much time it spent rendering. Rendering here means the time that the browser itself takes to render the DOM on-screen — React render() calls are included in the JS execution time.

If you are doing Redux, use the slider in ReduxDevTools to repeat and record only the actions suspected to cause a slow re-render.
Try to measure them one at a time.

If you see that a React function like batchUpdates has a long total-time but short self-time, it means that there may be a performance issues with your React components, perhaps because they are rendering too often.
If this is the case we will use ReactPerf.

### React Perf

Install the React Perf addon

```
npm install react-addons-perf
```

Import it into your project and expose it to the global context:

```
import Perf from 'react-addons-perf'
window.Perf = Perf
```

Once installed in your project, you can use the React Perf Chrome Extension to call Perf functions, or issue calls directly from the console.

The process is straight-forward:

1. Call Perf.start()
Take some actions in your application. This will be recorded by Perf (use ReduxDevTools slider if possible)
2. Call Perf.stop()
Now that the action has been recorded, you can call 3 useful functions:
3. printInclusive() — prints how much time is spent in each component
printExclusive() — prints how much time is spent doing rendering for each component (not including componentsWillMount, componentDidMount, props processing…)
printWasted() — prints how much time was wasted rendering components that didn’t actually change (rendering was done only in the VirtualDOM layer, and no changes occurred in the browser DOM).This is the most important function, as it will bring up components that should be pure and instances of the anti-patterns described above.
printOperations() — will print the actual browser DOM manipulation, this is handy only when the browser spends too much time rendering.

## Should I try to optimize?

react-dom is very fast by itself. You should only try to optimize once you have a problem that you can quantify with React Perf.

For example, consider a simple component like this:

```js
const Label = ({text}) => <div className='label'>{text}</div>
```

In this component, text probably never changes. Making it pure feels right, but this probably wont yield any noticeable improvement. Running shallowCompare will make about the same calculations as react-dom would, so by making the component pure you’ve just moved the processing to a different place.

So in other words, Don’t imagine a future problem, wait till react-dom is not good enough by itself, it usually is.

This article was brought to you by welldone-software, Modern Software Boutique. contact us for experts in Angular, Node, React, .NET, Cordova, Mobile and Cloud.
