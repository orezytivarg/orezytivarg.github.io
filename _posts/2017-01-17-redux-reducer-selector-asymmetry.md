---
id: redux-reducer-selector-asymmetry
title: Redux 리듀서와 셀렉터간의 비대칭성
category: Redux
---
[원문보기](http://randycoulman.com/blog/2016/09/20/redux-reducer-selector-asymmetry/)

# Redux 리듀서와 셀렉터간의 비대칭성

In my previous post, I talked about using actions, reducers, and selectors to encapsulate the Redux state tree. In that post, I showed an approach that works great for a single top-level reducer, but doesn’t address how to handle decomposed reducers. Let’s talk about that.

## What’s the Problem?

There are a number of ways to break up the Redux reducer into smaller pieces. Mark Erikson is working on a pull request for the Redux documentation that does a great job of explaining the options.

The most common approach is to use Redux’ combineReducers function to split the state into separate, decoupled “slices”, each of which is handled by a sub-reducer. For example:

combineReducers

```js
import calendarReducer from './calendarReducer'
import todosReducer from './todosReducer'
import usersReducer from './usersReducer'

export default combineReducers({
  calendar: calendarReducer,
  todos: todosReducer,
  users: usersReducer
})
```

In this example, we’re building up our single top-level reducer by combining other reducers that each handle their own section of the state tree. Each of calendarReducer, todosReducer, and usersReducer are reducers in their own right.

For each section of the state tree, we’ll also want to have some selectors so that we keep the shape of the state tree hidden from the rest of our application. How do we do that?

The FAQ in the Redux documentation says:

> It’s generally suggested that selectors are defined alongside reducers and exported, and then reused elsewhere (such as in mapStateToProps functions, in async action creators, or sagas) to colocate all the code that knows about the actual shape of the state tree in the reducer files.

If the selectors are defined alongside the reducers, and the reducers operate on a subset of the state tree, what should the selectors do?

## What Are Our Options?

We really only have two options for our selectors:

1. We should make them parallel the reducers. They should operate on the same restricted subset of the state tree that the reducers do.

2. We should make them “global”. That is, selectors should always expect to be given the root of the entire state tree.

## Which Option Is Best?

Let’s look at where we use selectors. We identified several places last time:

- In the mapStateToProps functions of our container components.

- In thunk action creators.

- In our reducer tests.

I thought of another place where we might use selectors since writing the previous post:

- In our reducers themselves. Sometimes, a reducer will need to look at data in another part of the state (sub-)tree in order to do its job. Even though reducers are already coupled to the shape of the tree, it’s often convenient to use an already-defined selector for this.

For each of these uses of selectors, which ones need to work with the global state tree, and which ones prefer to work with the local, restricted state tree?

mapStateToProps is always called with the global state tree. The getState function that is passed to thunk action creators similarly returns the global state tree. However, our reducers and reducer tests work with the local state tree.

That’s a 50-50 split, so that doesn’t help us much.

## Can’t We Do Both?

Is there a way we can have selectors that work with both local and global state?

There are a couple of ways to do this.

### Hybrid

The first is something I’ll call a “hybrid” approach.

In this approach, we define all of our selectors to work on their local section of the state tree.

At the parent level (i.e., alongside the main reducer), we define a selector to get us to that local state tree. When we need to call a selector with global state, we first apply the top-level selector and then the localized selector. That ends up looking like this:

Hybrid selectors

```js
// In todosReducer.js:
const allTodos = state => {
 // get todos from local state
}

// In appReducer.js:
const todosState = state => state.todos

// In a container somewhere:
import { allTodos } from './todosReducer'
import { todosState } from './appReducer'

const mapStateToProps = state => ({
  todos: allTodos(todosState(state))
})
```

This works, but it’s kind of tedious to have to constantly repeat this code everywhere. Can we do better?

## Delegation

In Dan Abramov’s video, [Redux: Colocating Selectors with Reducers](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers), he uses a delegation approach instead.

This approach is similar to the hybrid approach, but moves the composition of the local selector and the state slice selector into the appReducer file.

As above, todosReducer.js exports an allTodos selector that works on the local (todos-only) state. Then, in the main reducer file, we’d have something like this:

Making a Global Selector

```js
import * as fromTodos from './todosReducer'

export const allTodos = state => fromTodos.allTodos(state.todos)
```

We’re defining a global-state version of allTodos that extracts the todos sub-section of the state tree and then calls the local-state version of allTodos with it.

In the reducer and reducer specs, we’d import the local version of allTodos from todosReducer. But in containers and action creators, we’d import global version of allTodos from the main reducer file instead.

So, at the cost of an extra function definition, we have two versions of the same selector. The first version operates on local state, and the second is defined in terms of the first, but operates on global state.

The advantage to this approach is that all of the knowledge of the state shape is encapsulated in the appropriate place.

The local-state version of the selector knows about its part of the state tree, and is coupled to it appropriately. But it doesn’t know or care where in the global state tree it lives.

The global-state version of the selector knows about where the appropriate local state lives. But it doesn’t know or care what the structure of that section of the state tree looks like. It delegates that responsibility to the local-state selector.

This encapsulation is there for both the hybrid and delegation approaches. The advantage of the delegation approach is that we don’t have to keep repeating the local selectors and the state slice selectors. In fact, the client code doesn’t even need to know that the composition is happening at all.

## Problem Solved, Right?

I think the delegation approach is a good solution to the problem. Yes, it costs writing an extra version of each selector, but the encapsulation and flexibility is worth it to me.

If we decompose our state tree into many nested levels, then this gets more tedious, because each level of reducer has to redefine all of the selectors below it. At some point, this might become unmanageable.

In addition, the Redux FAQ talks about a few different ways to structure projects:

> - Rails-style: separate folders for “actions”, “constants”, “reducers”, “containers”, and “components”
> - Domain-style: separate folders per feature or domain, possibly with sub-folders per file type
> - "Ducks”: similar to domain style, but explicitly tying together actions and reducers, often by defining them in the same file

If we structure our application using a Rails-style approach, then yes, I think the problem is solved. Dan Abramov’s delegation approach is a good one, for all of the reasons I mentioned above. Even if we choose to put our selectors in separate files from our reducers, we can use this technique.

But if we want to use a Domain-style or Ducks approach, this technique runs into issues with circular dependencies.

I’ll spend more time on this structure in my next post.
