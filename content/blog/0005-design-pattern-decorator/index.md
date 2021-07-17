---
title: Decorator Pattern
excerpt: Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.
date: 2021-07-02
tags: [design-pattern, head-first-design-patterns, decorator-pattern]
slug: design-pattern-decorator
cover: cover.jpg
---

This is my note for Chapter 3 of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work?

## What is the Decorator Pattern?

Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/03-starbuzz).

## A Real Case in my Daily Work

High order component (HOC) uses the Decorator Pattern!

A higher-order component (HOC) is an advanced technique in React for reusing component logic. HOCs are not part of the React API, per se. They are a pattern that emerges from React’s compositional nature.

Concretely, a higher-order component is a function that takes a component and returns a new component.

```js
const EnhancedComponent = higherOrderComponent(WrappedComponent)
```

### Case 1 - Connect to Redux Store

I have used `React-Redux` to connect my react components to the redux store for a long time. And `connect()` function provided by `React-Redux` implements the Decorator Pattern (or HOC Pattern ✅).

It provides its connected component with the pieces of the data it needs from the store, and the functions it can use to dispatch actions to the store.

It does not modify the component class passed to it; instead, it returns a new, connected component class that wraps the component you passed in.

```js
import { login, logout } from "./actionCreators"

const mapState = state => state.user
const mapDispatch = { login, logout }

// first call: returns a hoc that you can use to wrap any component
const connectUser = connect(mapState, mapDispatch)

// second call: returns the wrapper component with mergedProps
// you may use the hoc to enable different components to get the same behavior
const ConnectedUserLogin = connectUser(Login)
const ConnectedUserProfile = connectUser(Profile)
```

### Case 2 - Authorization

There are several authorization levels for websites. So I need to find a way to determine whether a user can access the page. And I try to solve it with HOC which also implements the Decorator Pattern.

First, define a factory of auth HOC because there are several authorization levels.

```js
function authHOCFactory(allowedRole: string) {
  return function withAuthorization(WrappedPage: React.ComponentType) {
    return () => {
      // get permission from a custom usePermission hook
      const { data, isLoading } = usePermission()

      // show no permission page if the user is not allowed to access this page
      if (!data?.roles?.includes(allowedRole)) {
        return <NoPermissionPage />
      }

      return <WrappedPage />
    }
  }
}
```

Next, generate HOCs for different access levels.

```js
const requireAdminHOC = authHOCFactory(Role.ADMIN)
const requireDeveloperHOC = authHOCFactory(Role.DEVELOPER)
const requireEditorHOC = authHOCFactory(Role.EDITOR)
const requireMemberHOC = authHOCFactory(Role.MEMBER)
```

Finally, use those HOCs as decorators to wrap our page components. For example, only members can access the dashboard page.

```js
const DashboardPage = () => {
  return <div>Dashboard</div>
}

export default requireMemberHOC(DashboardPage)
```

## Reference

- https://reactjs.org/docs/higher-order-components.html
- https://react-redux.js.org/
