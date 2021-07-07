---
title: Frontend Milestones in JunyiAcademy
excerpt: Make the codebase more better and more maintainable is one of the most important tasks for a senior frontend engineers in JunyiAcademy.
date: 2021-04-20
tags: [junyiacademy]
slug: milestones-in-junyiacademy
cover: cover.jpg
---

Codebase of JunyiAcademy was forked from KhanAcademy in 2013. It is harder and harder for us to maintain and develop features because of the technical debt. Therefore, I know it's time to face the `Bootstrap`, `Jinja2`, `Handlebars`, `Backbone`, `jQuery`, etc as soon as I joined JunyiAcademy in May 2020.

## New Frontend Architecture

There had been 15% React code before I joined JunyiAcademy. So I doesn't spend time convincing the team to adapt it. What I need to do is provide a good frontend architecture and write the documents for everyone to follow.

The new tech stack is `React`, `React Query`, `Redux`, `Redux Toolkit`, `Testing Library`, `TypeScript`, `Next.js` and `Material UI`.

### SSR and Preview Mode

`Next.js` is chosen to be the new frontend server side rendering framework for the new architecture (previous one is `Flask` + `Jinja2` 🥸). It provides a lot of basic features such as SSR, SSG, image optimization, etc. And preview mode which is one of the advanced features can benefits our CMS a lot.

### Separate Server Side and Client Side State

`React Query` is a server-state library, responsible for managing asynchronous operations between your server and client. So we create our own custom hooks deal with server data with `useQuery` and `useMutation`.

`Redux` code becomes much cleaner because it's only a client state management. Without `React Query`, we use `Redux Observable` to handle those side effects caused by requests. But it's not necessary now, so the only thing stored in the `Redux` store is client side state.

### Efficient Redux Development

`Redux Toolkit` gives us the opportunity to remove those complicated configuration and boilerplate code for using `Redux`. `Slice` and `Selector` is the most frequent used API for us.

Slice reduces the boilerplate code like action creators, action types and reducers. And it can be given a specify name to prefix our actions. Make it easier to debug when you open the redux devTools.

Selectors are efficient and composable. Selectors can compute derived data, allowing Redux to store the minimal possible state. A selector is not recomputed unless one of its arguments changes. And They can be used as input to other selectors.

### Type Checking

`TypeScript` is great! It catch errors, free the memory of my brain and make the code more readable. So I'm very exciting to bring this technology to my team.

I create a new folder `interfaces` under `src` and define interfaces like `ITopic`, `IMenu`, `IBadge`, etc. Also, I use type guards to narrow down each of the api responses from unknown to specific interface.

In the future, I want to write unit tests for `TypeScript` like `tsd` and `dtslint` to protect our `TypeScript` code.

## Unit Testing

Unit Testing isn't taken seriously in JunyiAcademy for years. But it's very important. It can give us the confidence to not only do refactor but also develop new features.

In the beginning, there are some team members considering using `Enzyme` as our testing library. So I decide to give a dev sharing for comparison of `Enzyme` and `Testing Library`. I'm happy that our team decides to take `Testing Library`.

Some resource:

- [Why I use React Testing Library instead of Enzyme](https://blog.kewah.com/2019/testing-react-using-testing-library/)
- [Testing Implementation Details](https://kentcdodds.com/blog/testing-implementation-details)
- [Making your UI tests resilient to change](https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change)
- [Why I Never Use Shallow Rendering](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering)

## Development Experience

I believe user experience is not the only thing we should care about. Development experience is also very important. After all, better DX brings the better output. So I bring `Prettier`, develop CLI tools and publish a VSCode snippet for my team.

## Development Flow

There are a lot of great tools to improve the development flow. And here are what I choose: `Commitizen`, `Lint Staged`, `Husky`, `Semantic Release` and `jest-junit`. `Semantic Release` and `Commitizen` are integrated very well and easy to integrate with `GitLab` (although not that easy with `GitHub`). Very recommend you to try those great tools if your team can't afford a full-time operation engineer.
