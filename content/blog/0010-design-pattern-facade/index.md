---
title: Facade Pattern
excerpt: Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.
date: 2021-07-11
tags: [design-pattern, head-first-design-patterns, facade-pattern]
slug: design-pattern-facade
cover: cover.jpg
---

This is my notes for Chapter 7 (part 2) of [Head First Design Pattern, 2nd Edition](https://learning.oreilly.com/library/view/head-first-design/9781492077992/).

And where can this pattern be applied in my daily work?

## What is the Facade Pattern?

Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

## Book's Example in TypeScript

The book's example is written in Java. So I rewrite it in [TypeScript](https://github.com/wtlin1228/typescript-head-first-design-patterns-2nd-edition/tree/main/07-2-home-theater).

## A Real Case in my Daily Work

React custom hooks are widely used by react developers since 2018. Custom hooks with the Facade Pattern provide a straightforward way and keep our application more readable.

### Custom Hook - `useDropdown`

`useDropdown` returns the selected option, dropdown component and handle function for selecting option.

```js
const useDropdown = (label, defaultState, options) => {
  const [state, setState] = React.useState(defaultState)
  const id = `use-dropdown-${label.replace(" ", "").toLowerCase}`
  const Dropdown = () => (
    <div>
      <label htmlFor={id}>{label} </label>
      <select
        id={id}
        value={state}
        onChange={e => setState(e.target.value)}
        onBlur={e => setState(e.target.value)}
        disabled={options.length === 0}
      >
        <option>All</option>
        {options.map(item => (
          <option key={item} value={item}>
            {item}
          </option>
        ))}
      </select>
    </div>
  )

  return [state, Dropdown, setState]
}
```

### Use `useDropdown`

It's so easy to create multiple dropdowns with `useDropdown` custom hook.

```js
const dogOptions = ["MAX", "APOLLO", "MURPHY", "DIESEL", "TOBY"]
const catOptions = ["Luna", "Bella", "Cleo", "Nala", "Oscar"]

const App = () => {
  const [selectedDog, DogDropdown, setSelectedDog] = useDropdown(
    "select a dog",
    dogOptions[0],
    dogOptions
  )

  const [selectedCat, CatDropdown, setSelectedCat] = useDropdown(
    "select a cat",
    catOptions[0],
    catOptions
  )

  return (
    <div className="App">
      <h1>Hello Facade Pattern</h1>
      <h2>
        You selected {selectedDog} and {selectedCat}!
      </h2>

      <DogDropdown />
      <CatDropdown />

      <button
        onClick={() => {
          setSelectedDog(dogOptions[0])
          setSelectedCat(catOptions[0])
        }}
      >
        Reset
      </button>
    </div>
  )
}
```
