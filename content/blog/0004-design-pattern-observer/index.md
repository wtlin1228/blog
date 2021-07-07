---
title: Observer Pattern
excerpt: The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.
date: 2021-06-30
tags: [design-pattern, head-first-design-patterns, observer-pattern]
slug: design-pattern-observer
cover: cover.jpg
---

This is my notes for Chapter 2 of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work?

## What is the Observer Pattern?

The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/02-weather-station).

## A Real Case in my Daily Work

I have used `Redux` to handle global state for a long time. And `Redux` implements the Observer Pattern!

### Store

A store has `dispatch`, `subscribe` and `getState` methods. Store itself is actually implementing the subject interface. The notification is triggered by the subject instead of the client. And it's **pull model** because the subject doesn't send state to the observers.

- subject.attach(Observer) -> store.subscribe(listener)
- subject.detach(Observer) -> store.subscribe(listener) will return an unsubscribe function
- subject.setState(newState) -> store.dispatch(action)
- subject.notify() -> store.dispatch(action) will trigger notifications for all listeners
- subject.getState() -> store.getState()

### Example

1. Create a store

   ```js
   import { createStore } from "redux"

   const ADD = "ADD"
   const SUB = "SUB"

   const reducer = (state, action) => {
     switch (action.type) {
       case ADD:
         state.count += action.payload.count
         return state
       case SUB:
         state.count -= action.payload.count
         return state
       default:
         return state
     }
   }

   const store = createStore(reducer, {
     count: 0,
   })
   ```

1. Create observers and subscribe to the store

   ```js
   // observer1 will pull state from the store
   const observer1 = () => {
     console.log(store.getState())
   }

   const observer2 = () => {
     console.log("I just want to know the state has changed")
   }

   store.subscribe(observer1)
   store.subscribe(observer2)
   ```

1. Unsubscribe from the store

   ```js
   const unSubscribe = store.subscribe(someObserver)

   unSubscribe()
   ```

1. Dispatch actions and see the notifications

   ```js
   const addActionCreator = count => ({
     type: ADD,
     payload: { count },
   })

   const subActionCreator = count => ({
     type: SUB,
     payload: { count },
   })

   const increment = n => store.dispatch(addActionCreator(n))
   const decrement = n => store.dispatch(subActionCreator(n))

   increment(1)
   // observer1 says: { count: 1 }
   // observer2 says: I just want to know the state has changed

   increment(9)
   // observer1 says: { count: 10 }
   // observer2 says: I just want to know the state has changed

   decrement(4)
   // observer1 says: { count: 6 }
   // observer2 says: I just want to know the state has changed
   ```

### What's More

Two clients can share one Redux Store by lifting the Store to a backend service. Use Socket.IO to handle the communication between these two clients. This special implementation is extremely useful when we want to build a live customer service system. The customer service staff can use this service to show the customer how to do it. Or watch the customer's operating procedures then give feedback immediately.
