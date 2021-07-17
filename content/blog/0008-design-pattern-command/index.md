---
title: Command Pattern
excerpt: Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.
date: 2021-07-07
tags: [design-pattern, head-first-design-patterns, command-pattern]
slug: design-pattern-command
cover: cover.jpg
---

This is my note for Chapter 6 of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work?

## What is the Command Pattern?

Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/06-remote-control).

## A Real Case in my Daily Work

`Redux` is the first thought that came to me when I was reading the Command Pattern. The actions dispatched to store are actually the commands. So dispatch is the invoker and store is the receiver.

### Queue

Clients can pop actions from queues, then dispatch the actions on by one. For example, a client can get one action stored in the database through API then execute it.

### Log

The current state and the action that causes this error can be logged to the logging system like Sentry. Redux middleware is a good place to handle the error logging.

### Undoable

Actions can be undid and replay as many times as you want. This feature can be tested in the Redux DevTools easily.
