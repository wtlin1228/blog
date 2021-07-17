---
title: Iterator Pattern
excerpt: Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.
date: 2021-07-18
tags: [design-pattern, head-first-design-patterns, iterator-pattern]
slug: design-pattern-iterator
cover: cover.jpg
---

This is my note for Chapter 9 (part 1) of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work?

## What is the Iterator Pattern?

Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/09-1-menu).

## A Real Case in my Daily Work

I use `for/of` loop and spread syntax in JavaScript almost everyday. There are interfaces `Iterable` and `Iterator` in JavaScript. Iterable objects can be iterated with the `for/of` loop and be destructed with the spread syntax.

### Implement Iterable Object

Let's define a `foo` object with a data property.

```js
const foo = {
  data: [1, 2, 3, 4, 5],
}
```

Implement the `Iterable` interface to make the `foo` object iterable.

```js
const foo = {
  data: [1, 2, 3, 4, 5],
  [Symbol.iterator]() {
    let position = -1
    return {
      next: () => {
        position++
        return {
          done: position >= this.data.length,
          value: this.data[position],
        }
      },
    }
  },
}
```

Or implement it simply this way.

```js
const foo = {
  data: [1, 2, 3, 4, 5],
  [Symbol.iterator]() {
    return this.data[Symbol.iterator]()
  },
}
```

### Use Iterable Object

We can treat the `foo` object as an iterable since it implements the `Iterable` interface.

`for/of` loop

```js
for (let x of foo) {
  console.log(x)
} // 1, 2, 3, 4, 5
```

spread syntax

```js
const a = [...foo]
console.log(a) // [1, 2, 3, 4, 5]
```
