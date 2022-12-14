---
title: Empower Testing Library with Custom Query
excerpt: empower-testing-library-with-custom-query
date: 2022-12-15
tags: [testing, testing-library, custom-query]
slug: empower-testing-library-with-custom-query
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

[Testing Library](https://testing-library.com/) is simple and complete testing utilities that encourage good testing practices.
It provides many convenient queries such as `ByRole`, `ByLabelText`.
And it also gives us the ability to write our own custom query.

There are three ways to use those queries:

- render

  ```js
  const { getByText } = render(<App />)
  getByText("Paddle Roll")
  ```

- screen

  ```js
  render(<App />)
  screen.getByText("Cramp Roll")
  ```

- within

  ```js
  render(<App />)
  within(stage).getByText("Shim Sham Shimmy")
  ```

It won't be unfamiliar for frontend developers to wrap prices with thousand separators.

```js
const price = thousandSeparator("1234567")
console.log(price) // '1,234,567'
```

And you need to import the `thousandSeparator` function into your test every time you want to test any number that is thousand separated.

```jsx
import thousandSeparator from "some-where"

it("...", () => {
  // Arrange
  const price = "1234567"
  render(<Price price={price} />)

  // Assert
  const thousandSeparatedPrice = thousandSeparator(price)
  expect(screen.getByText(thousandSeparatedPrice)).toBeInTheDocument()
})
```

It works but the developer experience could be improved further.
So, let's write a custom query for it.

```js
// test-utils/custom-queries/byThousandSeparatedNumber.js

import { buildQueries, queryAllByText } from "@testing-library/react"
import thousandSeparator from "hq/report/utils/thousandSeparator"

const queryAllByThousandSeparatedNumber = (container, id, options) =>
  queryAllByText(container, thousandSeparator(id), options)

const getMultipleError = (container, thousandSeparatedNumber) =>
  `Found multiple elements with the number: ${thousandSeparatedNumber}`
const getMissingError = (container, thousandSeparatedNumber) =>
  `Unable to find an element with the number: ${thousandSeparatedNumber}`

const [
  queryByThousandSeparatedNumber,
  getAllByThousandSeparatedNumber,
  getByThousandSeparatedNumber,
  findAllByThousandSeparatedNumber,
  findByThousandSeparatedNumber,
] = buildQueries(
  queryAllByThousandSeparatedNumber,
  getMultipleError,
  getMissingError
)

export {
  queryByThousandSeparatedNumber,
  queryAllByThousandSeparatedNumber,
  getByThousandSeparatedNumber,
  getAllByThousandSeparatedNumber,
  findAllByThousandSeparatedNumber,
  findByThousandSeparatedNumber,
}
```

Then inject this new query into `render`, `screen` and `within` by making your own ones. Don't worry, just a few lines.

```js
// test-utils/index.js

import { render, queries, screen, within } from "@testing-library/react"
import * as customQueries from "./custom-queries"

const customScreen = {
  ...screen,
  ...within(document.body, customQueries),
}

const allQueries = {
  ...queries,
  ...customQueries,
}

const customRender = (ui, options) =>
  render(ui, { queries: allQueries, ...options })

const customWithin = element => within(element, allQueries)

// re-export everything
export * from "@testing-library/react"

// override render, screen and within
export {
  customScreen as screen,
  customRender as render,
  customWithin as within,
}
```

Now, you have created the `ByThousandSeparatedNumber` query and customized `render`, `screen`, `within`.
To be a good team player. Let's write some simple tests to verify them.

<!-- prettier-ignore-start -->
```js
// test-utils/custom-queries/__tests__/byThousandSeparatedNumber.test.js

import { render, screen, within } from "../../test-utils"

it("get*ByThousandSeparatedNumber", () => {
  const { 
    getByThousandSeparatedNumber, 
    getAllByThousandSeparatedNumber 
  } = render(
    <div data-fe-test-id="test-id">
      <p>12,345,678</p>
      <p>9,888,777,666</p>
      <p>9,888,777,666</p>
    </div>
  )

  expect(getByThousandSeparatedNumber(12345678)).toBeInTheDocument()
  expect(getAllByThousandSeparatedNumber(9888777666)).toHaveLength(2)

  expect(screen.getByThousandSeparatedNumber(12345678)).toBeInTheDocument()
  expect(screen.getAllByThousandSeparatedNumber(9888777666)).toHaveLength(2)

  const wrapper = screen.getByTestId("test-id")
  expect(within(wrapper).getByThousandSeparatedNumber(12345678)).toBeInTheDocument()
  expect(within(wrapper).getAllByThousandSeparatedNumber(9888777666)).toHaveLength(2)
})

it("query*ByThousandSeparatedNumber", () => {
  const { 
    queryByThousandSeparatedNumber, 
    queryAllByThousandSeparatedNumber 
  } = render(
    <div data-fe-test-id="test-id">
      Hello, ThousandSeparatedNumber Query
    </div>
  )

  expect(queryByThousandSeparatedNumber(12345678)).not.toBeInTheDocument()
  expect(queryAllByThousandSeparatedNumber(9888777666)).toHaveLength(0)

  expect(screen.queryByThousandSeparatedNumber(12345678)).not.toBeInTheDocument()
  expect(screen.queryAllByThousandSeparatedNumber(9888777666)).toHaveLength(0)

  const wrapper = screen.getByTestId("test-id")
  expect(within(wrapper).queryByThousandSeparatedNumber(12345678)).not.toBeInTheDocument()
  expect(within(wrapper).queryAllByThousandSeparatedNumber(9888777666)).toHaveLength(0)
})

it("find*ByThousandSeparatedNumber", async () => {
  const { 
    findByThousandSeparatedNumber, 
    findAllByThousandSeparatedNumber 
  } = render(
    <div data-fe-test-id="test-id">
      <p>12,345,678</p>
      <p>9,888,777,666</p>
      <p>9,888,777,666</p>
    </div>
  )

  expect(await findByThousandSeparatedNumber(12345678)).toBeInTheDocument()
  expect(await findAllByThousandSeparatedNumber(9888777666)).toHaveLength(2)

  expect(await screen.findByThousandSeparatedNumber(12345678)nTheDocument()
  expect(await screen.findAllByThousandSeparatedNumber(9888777666)eLength(2)

  const wrapper = screen.getByTestId("test-id")
  expect(await within(wrapper).findByThousandSeparatedNumber(12345678)).toBeInTheDocument()
  expect(await within(wrapper).findAllByThousandSeparatedNumber(9888777666)).toHaveLength(2)
})
```
<!-- prettier-ignore-end -->

If you structure your code this way. Next time you need a new custom query,
you could just create one under the `test-utils/custom-queries/` and
start using it in your tests right away. 🚀
