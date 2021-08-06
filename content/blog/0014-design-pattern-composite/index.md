---
title: Composite Pattern
excerpt: Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients test individual objects and compositions of objects uniformly.
date: 2021-07-21
tags: [design-pattern, head-first-design-patterns, composite-pattern]
slug: design-pattern-composite
cover: cover.jpg
---

This is my note for Chapter 9 (part 2) of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work?

## What is the Composite Pattern?

Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients test individual objects and compositions of objects uniformly.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/09-2-composite-menu).

## A Real Case in my Daily Work

Redux uses this pattern when combining multiple reducers together. As our app grows more complex, we'll want to split our reducing function into separate functions, each managing independent parts of the state. So we can use `combineReducers` to compose reducers into tree structures.

### `combineReducers`

The `combineReducers` helper function turns an object whose values are different reducing functions into a single reducing function you can pass to `createStore`.

```js
function combineReducers(reducers) {
  // ...
  return function combination(state, action) {
    // ...
    const nextState = {}
    Objects.entities(reducers).forEach(([key, reducer]) => {
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      nextState[key] = nextStateForKey
    })
    // ...
  }
}
```

### Example

Assume our reducer has the following shape:

```
rootReducer
├── fooReducer
└── animalReducer
    ├── catReducer
    ├── dogReducer
    └── birdReducer
```

And our state is:

```js
const state = {
  foo: {},
  animal: {
    cat: {},
    dog: {},
    bird: {},
  },
}
```

When an action is dispatched, the nested reducers will be called as following:

```
-> rootReducer(state, action)
    -> fooReducer(state.foo, action)
    -> animalReducer(state.animal, action)
        -> catReducer(state.animal.cat, action)
        -> dogReducer(state.animal.dog, action)
        -> birdReducer(state.animal.bird, action)
```

A new state is computed after calling each reducer with their state and action. Then the store will notify its subscribers that the state has been changed.
