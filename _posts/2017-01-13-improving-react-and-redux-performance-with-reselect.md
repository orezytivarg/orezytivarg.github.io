---
id: css-evolution-from-css-sass-bem-css-modules-to-styled-components
title: CSS의 진화 - CSS, SASS, BEM, CSS Modules로부터 Styled Components로
category: Redux
---
[원문보기](http://blog.rangle.io/react-and-redux-performance-with-reselect/)

# Improving React and Redux performance with Reselect

![](http://blog.rangle.io/content/images/2016/06/neil-fenton-memoization-1.gif)

When used together, React and Redux are an awesome combination of technologies that help us structure applications with a true separation of concerns. Even with React being extremely performant out of the box, there comes a time when even higher performance is required.

One of the more expensive operations that React can perform is the rendering cycle. When a component detects a change in input, the render cycle is triggered.

When we first get started with React, we typically don’t worry about how costly these render cycles are. But as our UIs grow in complexity, we need to start concerning ourselves with them. React offers us some tools to hijack the render cycle and prevent re-rendering if we deem it isn't necessary. To do this, we can tap into the componentShouldUpdate lifecycle event, and return a boolean that says whether or not the component should update. This is the basis of the PureRenderMixin, which compares the incoming props and state with the previous props and state, and returns false if equality passes.

This, coupled with Immutable datasets, provides us with a substantial performance improvement because we can easily determine if a component should re-render or not. Unfortunately, this only goes so far.

Consider the following problem: We are building a shopping cart, with 3 types of input:

- Items in the cart
- Quantity of those items
- Applicable tax (based on state or province)

The problem is that whenever the state of any of the inputs is modified (a new item is added, a quantity is changed, or the selected state is changed), everything will need to be recalculated and rerendered. You can see how this would be problematic if we had hundreds of items in our cart. Changing the tax percentage would trigger a recalculation of the items in the cart, but shouldn’t. The tax percentage is simply a change in the derived data. Only the total, and tax total, should change and trigger subsequent updates. Let’s look at how we can fix these problems.

## Reselect to the rescue

Reselect is a library for building memoized selectors. We define selectors as the functions that retrieve snippets of the Redux state for our React components. Using memoization, we can prevent unnecessary rerenders and recalculations of derived data which in turn will speed up our application.

Consider the following example:

![](http://blog.rangle.io/content/images/2016/06/image00-1.png)

If we had several hundred, or thousands of items, rerendering all the items in our cart would be costly even if only the tax-percentage changes. What if we implemented a search? Should we have to recalculate the items and their taxes over and over every time the user searched their cart? We can prevent these costly operations by moving them to use memoized selectors. With memoized selectors, if the state tree is large, we don’t have to worry about expensive calculations being performed every time the state changes. We can also add additional flexibility to our frontend by breaking these out into individual components.

Let’s look at a simple selector using Reselect:

![](http://blog.rangle.io/content/images/2016/06/image03-1.png)

In the above example, we’ve broken our cart item retrieval function into two functions. The first function (Line 3) will simply get all the items in the cart and the second function represents a memoized selector. Reselect exposes the createSelector API which allows us to build a memoized selector. What this means is that getItemsWithTotals will be calculated the first time the function runs. If the same function is called again, but the input (the result of getItems) has not changed, the function will simply return a cached calculation of the items and their totals. If the items are modified (e.g. an item is added, a quantity is changed, anything that manipulates the result of getItems), then the function will be executed again.

This is a powerful concept as it allows us to completely optimize which components should be rerendered, and when their derived state should be recalculated. This means we no longer have to worry about getItems – and subsequently the total cost of each item being calculated – when operations that don’t impact their state are performed.

We can continue this trend by creating selectors for all of our derived data. This includes the subtotal calculation, the total tax calculation, and the final total:

![](http://blog.rangle.io/content/images/2016/06/image01-1.png)

## Making use of a selector

With our selectors in place, let’s now look at how we can take advantage of the getItemsWithTotals selector in one of our components:

![](http://blog.rangle.io/content/images/2016/06/image02-1.png)

We now have a component that only understands the items in the cart. This is a nice approach because it does not have to concern itself with totals, subtotals, etc. While it is not the most useful component for reuse, it is a very performant component. Changes that it does not concern itself with (e.g. changes to the tax calculation), will not cause additional rerenders.

Applying this approach to the rest of the shopping cart means we will have a component that is responsible for displaying the subtotal, total, and tax calculation.

Making these optimizations early in your application means less work in the future when you need to correct performance problems. I recommend moving to using reselect as soon as possible. One of the major benefits of moving our selectors out of our components means that we can easily test these derived data calculations just as we would any other JavaScript function. We simply mock our Redux state and then check for the expected output based on the state provided.

For a further demonstration of these concepts, refer to the demo here: https://github.com/neilff/react-redux-performance

# React & Redux Resources

For more on React, from our team to yours, you can check out these informative videos and webinars. Better yet, we encourage you to inquire about custom training to ramp up your knowledge of the fundamentals and best practices with custom course material designed and delivered to address your immediate needs. We also have more on redux [here](http://rangle.io/resources/tags/redux/). 
