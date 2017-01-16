---
id: encapsulating-the-redux-state-tree
title: Redux 상태 트리 캡슐화하기
category: Redux
---
[원문보기](http://randycoulman.com/blog/2016/09/13/encapsulating-the-redux-state-tree/)

# Redux 상태 트리 캡슐화하기

In a Redux application, the bulk of the application’s data is stored as a “state tree” in a central location, the store. The shape and structure of the state tree has a large impact on the ease of development and performance of the application. It is often valuable to refactor the state tree over time to address issues. How do we do this safely?

I’ve written about Redux a few times. Feel free to review those posts if you need an introduction or refresher.

When we need to make it safe to change a data structure, the most common tool to reach for is encapsulation, also known as data-hiding or information-hiding. The basic idea is that, rather than allowing direct access to the data structure, we provide an interface to the data instead.

That’s what we’ll do here.

## How Is the State Tree Used?

In order to encapsulate the state tree, we need to figure out how it is used in our application. With most data structures, there are two kinds of uses: reading and writing.

This is true in a Redux application as well. Because Redux is an implementation of the Flux architecture, it follows the principle of unidirectional data flow. That makes things a bit easier to reason about.

### Writing

We’ll start with the writing side of the equation, because that’s the most obvious in Redux.

Changes to the state tree are made by the reducer in response to various actions. The changes are not made directly; rather, the reducer returns a new state tree containing the desired changes. The Redux store takes care of remembering the new state tree.

### Reading

Reading the state tree tends to be more distributed in most Redux applications. Each container component implements a mapStateToProps function that reads parts of the state tree and injects those parts into its wrapped component as props.

Less obvious uses of the state tree are in action creators that work with the thunk middleware. Thunk actions are passed the store’s getState function and can use that function to access the state when creating actions.

A third place where we might read the state tree is in our reducer tests. They often need to know that the reducer has handled an action correctly.

## Now What?

Now that we know how the state tree is used, what do we do about it?

First, we can notice that the combination of the reducer and action creators already creates a layer of encapsulation on the writing side. Our applications don’t directly manipulate the state tree; rather, they dispatch actions that are handled by the reducer to manipulate the state tree for us. There’s nothing more we need to do here.

The reading side is less well-defined. Because reads are scattered through various container components, thunk actions, and tests, they’re harder to update when we need to make a change to our state shape.

An emerging best practice is to introduce an abstraction layer called selectors.

## Selectors

A selector is a function that takes the state and optional additional parameters and returns some data.

Rather than writing state.todos everywhere, we instead define a selector like allTodos(state) that returns all of the todos. That might seem like a pointless exercise, but it is critical to encapsulating the state tree.

By using this selector, none of the rest of the application knows or cares how the todos are stored in the state tree, making it much easier for us to refactor the state tree as needed.

Selectors can return data directly from the state tree, or they can perform some computation on the data before returning it.

The Redux documentation doesn’t talk much about selectors as a core concept in Redux, though they are mentioned in the section on Computing Derived Data, along with the excellent reselect library.

They are covered more thoroughly in Dan Abramov’s second series of Redux videos, particularly in Redux: Colocating Selectors with Reducers. If you haven’t watched these videos yet, I highly recommend them. As with Dan’s first series of videos, they are excellent.

## Are We There Yet?

With the reducer and actions encapsulating the writes and selectors handling the reads, are we done? Is it safe to refactor our state tree now?

For the most part, yes, we’re done. If all of our code makes use of actions and selectors and never accesses the state directly, then we know exactly what needs to change when we need to refactor our state tree.

Dan Abramov even suggests that the selectors live in the same file as the reducer to make this encapsulation more clear. I haven’t tried that option, but I definitely understand the reasoning behind it.

But what about the tests? Are they using the selectors?

## Testing

I wrote about Testing Redux Applications earlier. In the section on testing reducers, I showed the following test, adapted from the Redux documentation.

Reducer Tests

```js
import { expect } from 'chai'
import reducer from 'reducers/todos'
import ActionTypes from 'constants/ActionTypes'

describe('todos reducer', () => {
  it('initializes the state', () => {
    expect(reducer(undefined, {})).toEqual([])
  })

  it('adds a todo', () => {
    expect(reducer([], addTodo('Run the tests'))).toEqual([
      {
        text: 'Run the tests',
        completed: false,
        id: 0
      }
    ])
  })
})
```

Note that there are several places in this test that know about the shape of the state tree. Of all the places that could be coupled to the state tree shape, this one is the easiest to defend. But can we do better?

I’ve since evolved my reducer testing strategy a bit, and on my current project I’m experimenting with a complete decoupling of the tests from the state tree structure.

I start out by defining a const in the test for the initial state as generated by the reducer.

Initial State

```js
describe('todos reducer', () => {
  const initialState = reducer(undefined, {})

  # ...
})
```

If there’s anything important about the initial state, I’ll write some tests for it using my selectors. I make sure to write the tests from the point of view of the client of the state tree, and not in terms of the reducer implementation.

Testing the Initial State

```js
# ...

  describe('initial state', () => {
    it('starts with no todos', () => {
      expect(allTodos(initialState)).to.be.empty  
    })
  })

# ...
```

Note that this test says nothing about how the todos are stored or what the shape of the initial state actually is. It just checks something important about the observable properties of the state.

In order to test the handling of various actions, I’ll write tests something like the following:

Testing an Action

```js
# ...

  describe('adding a todo', () => {
    const state = initialState
    const action = addTodo('TODO')
    const newState = reducer(state, action)
    const addedTodo = allTodos(newState)[0]

    it('includes the todo in the list', () => {
      expect(addedTodo).to.exist
    })

    it('remembers the name', () => {
      expect(addedTodo.name).to.eq('TODO')
    })

    it('assigns the next available id', () => {
      expect(addedTodo.id).to.eq(0)
    })

    it('marks the todo as initially incomplete', () => {
      expect(addedTodo.complete).to.be.false
    })
  })

# ...
```

The way I’ve written this is somewhat verbose; it would be fine to check all of the properties of the added todo in one test instead. I tend to prefer the separate it blocks because the descriptions give me a place to communicate my intentions and to make it clear what responsibilities the reducer has.

I tend to keep the const state = ...; const action = ...; const newState = ... pattern, though. The regular structure of the tests make them easier to read.

If I’m writing a test that needs something other than the initial state, I’m experimenting with using actions to get the state I need.

For example, if I wanted to test a todo list with an already-existing todo, I’d start my test this way:

Starting With Other State

```js
# ...

  const state = reducer(initialState, addTodo('Pre-existing'))

# ...
```

If I need more than one action to get the starting state right, I’ll use reduce with an array of actions. Here I’m using the version of reduce from Ramda, but there are other options as well.

Multiple Actions

```js
# ...
  const state = reduce(reducer, initialState, [
    addTodo('Already complete'),
    completeTodo(0),
    addTodo('Still outstanding')  
  ])
# ...
```

So far, I really like the decoupling this gives me, but the jury is still out on whether this style of state setup is understandable enough for future readers of the code.

## What About Nested Reducers?

Everything I’ve talked about so far works great when we only have a single top-level reducer. The selectors and reducer play well together, and the testing approach works like a charm.

In most Redux applications, however, we use combineReducers to break the top-level reducer into sub-reducers, each working on a small portion of the state tree. But our selectors have to work with the entire state tree.

How do we deal with this asymmetry between reducers and selectors?

I’ll save the answer to that question for next week’s post.

## Conclusion

I highly recommend encapsulating Redux state by adding selectors along with the actions and reducers that we’ve already got.

Taking it a step further and writing reducer tests using only actions and selectors seems like a promising approach so far. I’ve been able to refactor my state tree by changing only the reducer and selectors. Notably, I didn’t have to change anything in the associated reducer test.
