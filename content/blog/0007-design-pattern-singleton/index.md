---
title: Singleton Pattern
excerpt: Ensure a class only has one instance, and provide a global point of access to it.
date: 2021-07-05
tags: [design-pattern, head-first-design-patterns, singleton-pattern]
slug: design-pattern-singleton
cover: cover.jpg
---

This is my notes for Chapter 5 of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work.

## What is Singleton Pattern?

Ensure a class only has one instance, and provide a global point of access to it.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/05-chocolate-factory).

## A Real Case in my Daily Work

I use Singleton Pattern to create a unique redux store which will be pass into different react-redux providers. The primary reason for having serval react-redux provider is that I'm migrating an non-react frontend codebase to react. Therefore, I need to render the react components into different containers during my migrating journey.

By the way, this pattern can also be used when the react-query client needs to be passed into many react-query providers.

### Multiple Stores or Single Store

As with several other questions, it is possible to create multiple distinct Redux stores in a page, but the intended pattern is to have only a single store. Having a single store enables using the Redux DevTools, makes persisting and rehydrating data simpler, and simplifies the subscription logic.

Some valid reasons for using multiple stores in Redux might include:

- Solving a performance issue caused by too frequent updates of some part of the state, when confirmed by profiling the app.
- Isolating a Redux app as a component in a bigger application, in which case you might want to create a store per root component instance.

However, creating new stores shouldn't be your first instinct, especially if you come from a Flux background. Try reducer composition first, and only use multiple stores if it doesn't solve your problem.

Similarly, while you can reference your store instance by importing it directly, this is not a recommended pattern in Redux. If you create a store instance and export it from a module, it will become a singleton. This means it will be harder to isolate a Redux app as a component of a larger app, if this is ever necessary, or to enable server rendering, because on the server you want to create separate store instances for every request.

### Example

Here is a simple bookstore app that users can read the books and recommend new books.

User stories:

1. When a user recommends a new book from `UserProfile`, this user will see the book appears in the `Bookstore`.
1. When a user chooses a new book from `Bookstore`, this user will see the current reading book is updated in the `UserProfile`.

First, define the Store class with Singleton Pattern.

```js
// src/store.js

import { configureStore } from "@reduxjs/toolkit"
import { rootReducer } from "./slices/rootReducer"

class Store {
  static uniqueStore

  static getStore() {
    if (!Store.uniqueStore) {
      Store.uniqueStore = Store.generateStore()
    }

    return Store.uniqueStore
  }

  static generateStore() {
    return configureStore({
      reducer: rootReducer,
    })
  }
}

export default Store
```

After defining the Store class with Singleton Pattern, use `Store.getStore()` to get the unique store instance. And this unique store can be pass to as many as provider as you want without sacrificing the benefits described above.

```js
// src/index.js

ReactDOM.render(
  <React.StrictMode>
    <Provider store={Store.getStore()}>
      <UserProfile />
    </Provider>
  </React.StrictMode>,
  document.getElementById("user-profile")
)

ReactDOM.render(
  <React.StrictMode>
    <Provider store={Store.getStore()}>
      <Bookstore />
    </Provider>
  </React.StrictMode>,
  document.getElementById("bookstore")
)
```

The complete example is in this github repository - https://github.com/wtlin1228/redux-with-singleton-pattern. Try to replace `Store.getStore()` with `Store.generateStore()` to see the differences.

## Reference

- [Can or should I create multiple stores? Can I import my store directly, and use it in components myself?](https://redux.js.org/faq/store-setup#can-or-should-i-create-multiple-stores-can-i-import-my-store-directly-and-use-it-in-components-myself)
