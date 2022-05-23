---
title: WIP - Summary of TypeScript Congress 2022
excerpt:
date: 2022-05-22
tags: [conference, typescript, types]
slug: summary-of-typescript-congress-2022
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

This is my note for [TypeScript Congress 2022](https://typescriptcongress.com/).

- [TypeScript and the Database: Who Owns the Types?](#typescript-and-the-database-who-owns-the-types)
- [Lessons from Maintaining TypeScript Libraries](#lessons-from-maintaining-typescript-libraries)
- [Type Safety At Runtime in TypeScript](#type-safety-at-runtime-in-typescript)
- [Onboarding React Developers To TypeScript](#onboarding-react-developers-to-typescript)
- [TypeScript for Library Authors: Harnessing the Power of TypeScript for DX](#typescript-for-library-authors-harnessing-the-power-of-typescript-for-dx)
- [Understanding types as sets](#understanding-types-as-sets)
- [Alternatives to TypeScript](#alternatives-to-typescript)
- [How to build distributed systems in TypeScript](#how-to-build-distributed-systems-in-typescript)
- [Writing universal modules for Deno, Node, and the browser](#writing-universal-modules-for-deno-node-and-the-browser)
- [Plug-in architecture: how TypeScript let us paint-by-numbers](#plug-in-architecture-how-typescript-let-us-paint-by-numbers)
- [How to properly handle URL slug changes in Next.js](#how-to-properly-handle-url-slug-changes-in-nextjs)

---

# TypeScript and the Database: Who Owns the Types?

Recommend: ❤️❤️❤️❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/typescript-and-the-database-who-owns-the-types)

# Lessons from Maintaining TypeScript Libraries

Recommend: ❤️❤️❤️❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/lessons-from-maintaining-typescript-libraries)

# Type Safety At Runtime in TypeScript

# Onboarding React Developers To TypeScript

# TypeScript for Library Authors: Harnessing the Power of TypeScript for DX

# Understanding types as sets

Titian-Cornel Cernicova-Dragomir introduced the concept of variance as in pertains to generic types.

🔖 I love the way Titian-Cornel showing us that type is a set of values that a variable can process.

---

Recommend: ❤️❤️❤️❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/understanding-types-as-sets)

---

## Set operations

### Unions of literal types

Literal types become more useful when paired with the set operating union.

```ts
type Yes = "Yes"
type One = 1
type True = true

type Truthy = Yes | One | True
```

### Intersections of primitive types

What is the intersection of string and number?

```ts
type IsInBothStringAndNumber = string & number // it's never
```

### Filtering a union of primitives

How can we extract string literal types from this union?

```ts
type BooleanLike = "Yes" | "No" | 0 | 1 | true | false

type JustString = BooleanLike & string
// type JustString = "Yes" | "No"

type JustNumber = BooleanLike & number
// type JustNumber = 0 | 1

type JustBoolean = BooleanLike & boolean
// type JustBoolean = boolean
```

### Union and intersection reduction

TypeScript will use distributivity to bring combinations of union/intersection to a canonical form:

`type T = A & (B | C) <=> type T = (A & B) | (A & C)`

```ts
type JustStrings
  = BooleanLike & string
  = ("Yes" | "No" | 0 | 1 | true | false) & string
  = ("Yes" & string) | ("No" & string) | (0 & string) | (1 & string) | (true & string) | (false & string)
  = "Yes" | "No" | never | never | never | never
  = "Yes" | "No"
```

## Object types

Object types define the requirements for an object.

```ts
type Person = { name: string }
function withPerson(p: Person) {
  console.log(p.name)
}

withPerson({}) // type error

withPerson({ name: "Leo" }) // valid

// Excess property checks are the exception to this.
let p1 = { name: "John", age: 18 }
withPerson(p1) // valid
withPerson({ name: "John", age: 18 }) // type error on age
```

### Consequences of object types

```ts
type Person = { name: string }
function withPerson(p: Person) {
  for (const key of Object.keys(p)) {
    console.log(p[key].toUpperCase())
    // type error on key:
    // Element implicitly has an 'any' type because expression
    // of type 'string' can't be used to index type 'Person'.
    // No index signature with a parameter of type 'string' was
    // found on type 'Person'.
  }
}
```

We might think it's a bug and TypeScript will fix it. So we might want to resolve the type error by the following workaround:

```ts
type Person = { name: string }
function withPerson(p: Person) {
  for (const key of Object.keys(p) as Array<keyof Person>) {
    console.log(p[key].toUpperCase())
  }
}

withPerson({ name: "John" }) // valid

let p1 = { name: "John", age: 18 }
withPerson({ p1 }) // no type error, but runtime error
```

### Unions of Object Types

```ts
type Person = { name: string; description?: string }
type EntityWithId = { id: string; description?: string }

let p: Person | EntityWithId = Math.random() > 0.5 ? { name: "" } : { id: "" }

console.log(p.name) // type error on name
console.log(p.id) // type error on id
console.log(p.description) // valid
```

### Intersections of Object Types

```ts
type Person = { name: string }
type EntityWithId = { id: string }

let p: Person & EntityWithId = { name: "", id: "" }

console.log(p.name) // valid
console.log(p.id) // valid
```

### The confusing part about unions and intersections

- Union of object types:
  - Can hold the union of values form constituent types
  - Allow access to an **intersection** of members
- Intersection of object types:
  - Can hold values in the intersection of constituent types
  - Allow access to a **union** of members

### Filtering a union of objects

- Use a conditional
- Intersections also do the job
  - Mostly 😁

```ts
type Shape =
  | { type: "circle", radius: number }
  | { type: "square", radius: number }
type Circle
  = { type: "circle" } & Shape
  = { type: "circle"} & (
      { type: "circle", radius: number } |
      { type: "square", radius: number }
    )
  = ({ type: "circle"} & { type: "circle", radius: number }) |
    ({ type: "square"} & { type: "circle", radius: number })
  = ({ type: "circle"} & { type: "circle", radius: number }) |
    ({ type: never, radius: number})
  = ({ type: "circle"} & { type: "circle", radius: number }) | never
  = ({ type: "circle"} & { type: "circle", radius: number })
  = { type: "circle", radius: number }
```

## Base type

What is a base type when we talk about sets?

### For other typed languages

- Again, our intuition is shaped by nominally typed languages.
- We can check the type definition:
  - For extends clauses (or equivalent)
  - For implements clauses (or equivalent)

C++

```c++
class Animal {
public:
    virtual void speak() {}
};

class Dog: public Animal {
public:
    void speak() { cout << "Bark"; }
};
```

Java

```java
class Animal {
    public void speak() {}
}

class Dog extends Animal {
    public void speak() { System.out.println("Bark"); }
}
```

C#

```c#
class Animal {
    public virtual void speak() {}
}

class Dog:Animal {
    public override void speak() { console.WriteLine("Bark"); }
}
```

### For TypeScript

- What does it mean to have a variable typed as a base type?
- What can a variable of type Animal hold:
  - It can hold an Animal `let a1: Animal = new Animal()`
  - It can hold a Dog `let a2: Animal = new Dog()`
  - It can also hold a Cat `let a3: Animal = new Cat()`
  - It can hold any subtype of Animal `let a4: Animal = new SiameseCat()`

Generally, a variable typed as a base type can hold an instance of the type, or any subtype.

- A base type describes a set that contains all subtype values
- A subtype is always a subset of the base type

That is to say, a type can have unlimited base types.

# Alternatives to TypeScript

Ashley Claymore compared TypeScript with Flow.js and Hegel, reviewed the similarities and differences among the three.

🔖 I'm not that familiar with TypeScript enough to see the value of this talk.

---

Recommend: ❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/alternatives-to-typescript)

# How to build distributed systems in TypeScript

Loren Sands-Ramshaw showed us why should we use `temporal` to build distributed systems.

🔖 It's amazing! Temporal makes reliability much easier!

---

Recommend: ❤️❤️❤️❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/how-to-build-distributed-systems-in-typescript)

---

## 1. POST /orders

```ts
import type { VercelRequest, VercelResponse } from "@vercel/node"
import { decodeJWT } from "../auth"
import {
  fulfillmentService,
  inventoryService,
  paymentService,
} from "../services"

export default async (request: VercelRequest, response: VercelResponse) => {
  const { itemId, quantity, addressId } = request.body
  const { userId } = decodeJWT(request.headers.authorization)

  await inventoryService.reserve({ itemId, quantity })
  await paymentService.charge({ userId, itemId, quantity })
  await fulfillmentService.sendPackage({ itemId, quantity, addressId })

  response.status(200).send(`Order submitted!`)
}
```

## 2. Make it reliable

We would like to refund the charge and cancel the reservation of inventory if fulfillment failed.

```ts
import type { VercelRequest, VercelResponse } from "@vercel/node"
import { decodeJWT } from "../auth"
import {
  fulfillmentService,
  inventoryService,
  paymentService,
} from "../services"

export default async (request: VercelRequest, response: VercelResponse) => {
  const { itemId, quantity, addressId } = request.body
  const { userId } = decodeJWT(request.headers.authorization)

  const reservation = await inventoryService.reserve({ itemId, quantity })
  if (reservation.failed) {
    response.status(400).send(`Don't have enough inventory`)
    return
  }

  const payment = await paymentService.charge({ userId, itemId, quantity })
  if (payment.failed) {
    await inventoryService.unreserve({ itemId, quantity })
    response.status(400).send(`Payment failed`)
    return
  }

  const fulfillment = await fulfillmentService.sendPackage({
    itemId,
    quantity,
    addressId,
  })
  if (fulfillment.failed) {
    await paymentService.refund({ userId, itemId, quantity })
    await inventoryService.unreserve({ itemId, quantity })
    response.status(400).send(`Can't ship to your address`)
    return
  }

  response.status(200).send(`Order submitted!`)
}
```

### Next: add retries and timeouts

```ts
// ...emit...

async function retry<Result>(
  serviceCall: () => Promise<Result>,
  maxAttempts = 10,
  callTimeout = 30 * 1000,
  initialInterval = 1000
): Promise<Result> {
  let result: Result | undefined
  let error: any
  for (let attempt = 0; attempt < maxAttempts; attempt++) {
    try {
      result = await Promise.race([
        serviceCall(),
        new Promise(resolve => {
          setTimeout(resolve, callTimeout, undefined)
        }),
      ])
      const timedOut = result === undefined
      if (timedOut) {
        throw new Error("Timed out")
      }
      break
    } catch (e) {
      error = e
      await new Promise(resolve =>
        setTimeout(resolve, initialInterval * Math.pow(2, attempt))
      )
    }
  }
  if (!result) {
    throw error
  }
  return result
}

export default async (request: VercelRequest, response: VercelResponse) => {
  // ...emit...

  const reservation = await retry(() =>
    inventoryService.reserve({ itemId, quantity })
  )
  // ...emit...

  const payment = await retry(() =>
    paymentService.charge({ userId, itemId, quantity })
  )
  // ...emit...

  const fulfillment = await retry(() =>
    fulfillmentService.sendPackage({
      itemId,
      quantity,
      addressId,
    })
  )
  // ...emit...
}
```

### Next: idempotency

```ts
// ...emit...

export default async (request: VercelRequest, response: VercelResponse) => {
  const { itemId, quantity, addressId, requestId } = request.body
  const { userId } = decodeJWT(request.headers.authorization)

  const reservation = await retry(() =>
    inventoryService.reserve({ itemId, quantity, requestId })
  )
  if (reservation.failed) {
    response.status(400).send(`Don't have enough inventory`)
    return
  }

  const payment = await retry(() =>
    paymentService.charge({ userId, itemId, quantity, requestId })
  )
  if (payment.failed) {
    await inventoryService.unreserve({ itemId, quantity, requestId })
    response.status(400).send(`Payment failed`)
    return
  }

  const fulfillment = await retry(() =>
    fulfillmentService.sendPackage({
      itemId,
      quantity,
      addressId,
      requestId,
    })
  )
  if (fulfillment.failed) {
    await paymentService.refund({ userId, itemId, quantity, requestId })
    await inventoryService.unreserve({ itemId, quantity, requestId })
    response.status(400).send(`Can't ship to your address`)
    return
  }

  response.status(200).send(`Order submitted!`)
}
```

### Next: retry failure compensation

```ts
// ...emit...

export default async (request: VercelRequest, response: VercelResponse) => {
  // ...emit...

  if (payment.failed) {
    await retry(() =>
      inventoryService.unreserve({ itemId, quantity, requestId })
    )
    response.status(400).send(`Payment failed`)
    return
  }

  // ...emit...
  if (fulfillment.failed) {
    await retry(() =>
      paymentService.refund({ userId, itemId, quantity, requestId })
    )
    await retry(() =>
      inventoryService.unreserve({ itemId, quantity, requestId })
    )
    response.status(400).send(`Can't ship to your address`)
    return
  }

  response.status(200).send(`Order submitted!`)
}
```

### Next: catch errors

```ts
// ...emit...

export default async (request: VercelRequest, response: VercelResponse) => {
  const { itemId, quantity, addressId, requestId } = request.body
  const { userId } = decodeJWT(request.headers.authorization)

  let error: any
  const reservation = await retry(() =>
    inventoryService.reserve({ itemId, quantity, requestId })
  ).catch(e => {
    error = e
  })
  if (!reservation || reservation.failed) {
    response
      .status(400)
      .send(reservation ? `Don't have enough inventory` : error.message)
    return
  }

  const payment = await retry(() =>
    paymentService.charge({ userId, itemId, quantity, requestId })
  ).catch(e => {
    error = e
  })
  if (!payment || payment.failed) {
    await retry(() =>
      inventoryService.unreserve({ itemId, quantity, requestId })
    )
    response.status(400).send(payment ? `Payment failed` : error.message)
    return
  }

  const fulfillment = await retry(() =>
    fulfillmentService.sendPackage({
      itemId,
      quantity,
      addressId,
      requestId,
    })
  ).catch(e => {
    error = e
  })
  if (!fulfillment || fulfillment.failed) {
    await retry(() =>
      paymentService.refund({ userId, itemId, quantity, requestId })
    )
    await retry(() =>
      inventoryService.unreserve({ itemId, quantity, requestId })
    )
    response
      .status(400)
      .send(fulfillment ? `Can't ship to your address` : error.message)
    return
  }

  response.status(200).send(`Order submitted!`)
}
```

### Next: save state

```ts
// ...emit...

import { MongoClient } from "mongodb"

const mongo = new MongoClient(`mongodb://localhost`)

// ...emit...

enum OrderState {
  CREATED,
  RESERVED,
  FAILED_TO_RESERVE,
  PAID,
  FAILED_TO_CHARGE,
  FAILED_TO_CHARGE_UNRESERVED,
  FULFILLED,
  FAILED_TO_FULFILL,
  FAILED_TO_FULFILL_UNRESERVED,
  FAILED_TO_FULFILL_REFUNDED,
}

export default async (request: VercelRequest, response: VercelResponse) => {
  const { itemId, quantity, addressId, requestId } = request.body
  const { userId } = decodeJWT(request.headers.authorization)

  try {
    await mongo.connect()
    const db = mongo.db(`orders`)
    const orders = db.collection(`orders`)
    const result = await orders.insertOe({
      itemId,
      quantity,
      addressId,
      requestId,
      userId,
      state: OrderState.CREATED,
    })
    if (!result.acknowledged) {
      response.status(500).send(`Failed to initiate order`)
      return
    }
    const _id = result.insertedId

    let error: any
    const reservation = await retry(() =>
      inventoryService.reserve({ itemId, quantity, requestId })
    ).catch(e => {
      error = e
    })
    if (!reservation || reservation.failed) {
      await orders.updateOne(
        { _id },
        { $set: { state: OrderState.FAILED_TO_RESERVE } }
      )
      response
        .status(400)
        .send(reservation ? `Don't have enough inventory` : error.message)
      return
    }
    await orders.updateOne(
      { _id },
      { $set: { state: OrderState.FAILED_TO_RESERVED } }
    )

    const payment = await retry(() =>
      paymentService.charge({ userId, itemId, quantity, requestId })
    ).catch(e => {
      error = e
    })
    if (!payment || payment.failed) {
      await orders.updateOne(
        { _id },
        { $set: { state: OrderState.FAILED_TO_CHARGE } }
      )
      await retry(() =>
        inventoryService.unreserve({ itemId, quantity, requestId })
      ).catch(() =>
        orders.updateOne(
          { _id },
          { $set: { state: OrderState.FAILED_TO_UNRESERVED } }
        )
      )
      response.status(400).send(payment ? `Payment failed` : error.message)
      return
    }
    await orders.updateOne({ _id }, { $set: { state: OrderState.PAID } })

    const fulfillment = await retry(() =>
      fulfillmentService.sendPackage({
        itemId,
        quantity,
        addressId,
        requestId,
      })
    ).catch(e => {
      error = e
    })
    if (!fulfillment || fulfillment.failed) {
      await orders.updateOne(
        { _id },
        { $set: { state: OrderState.FAILED_TO_FULFILL } }
      )
      await retry(() =>
        paymentService.refund({ userId, itemId, quantity, requestId })
      ).catch(() =>
        orders.updateOne(
          { _id },
          { $set: { state: OrderState.FAILED_TO_FULFILL_REFUNDED } }
        )
      )
      await retry(() =>
        inventoryService.unreserve({ itemId, quantity, requestId })
      ).catch(() =>
        orders.updateOne(
          { _id },
          { $set: { state: OrderState.FAILED_TO_FULFILL_UNRESERVED } }
        )
      )
      response
        .status(400)
        .send(fulfillment ? `Can't ship to your address` : error.message)
      return
    }
    await orders.updateOne({ _id }, { $set: { state: OrderState.FULFILLED } })

    response.status(200).send(`Order submitted!`)
  } catch (e: any) {
    response.status(500).send(`Internal server error: ${e?.message}`)
  } finally {
    await mongo.close()
  }
}
```

### Next: db retries & error handling, worker: updatedAt. returning earlier.

We don't have time to do it 😂

## 3. Simplify with Temporal SDK

Use Temporal

```ts
import {
  WorkflowClient,
  WorkflowExecutionAlreadyStartedError,
} from "@temporalio/client"
import type { VercelRequest, VercelResponse } from "@vercel/node"
import { decodeJWT } from "../auth"
import {
  fulfillmentService,
  inventoryService,
  paymentService,
} from "../services"

const client = new WorkflowClient()

export default async (request: VercelRequest, response: VercelResponse) => {
  const { itemId, quantity, addressId, requestId } = request.body
  const { userId } = decodeJWT(request.headers.authorization)

  try {
    await client.start(processOrder, {
      args: [{ itemId, quantity, addressId, userId }], // type inference works!
      workflowId: requestId,
      taskQueue: "my-online-store",
    })
    response.status(200).send(`Order submitted!`)
  } catch (e: any) {
    if (e instanceof WorkflowExecutionAlreadyStartedError) {
      response.status(400).send(`Order already submitted`)
    } else {
      response
        .status(500)
        .send(
          `Unknown error. Please try again later. Error message: ${e?.message}`
        )
    }
  }
}
```

```ts
import { proxyActivities } from "@temporalio/workflow"
import type * as activities from "./activities"
import { Order } from "./types"

const { reserve, unreserve, charge, refund, sendPackage } = proxyActivities<
  typeof activities
>({
  startToCloseTimeout: `1 minute`,
})

export async function processOrder(order: Order): Promise<void> {
  await reserve(order)

  try {
    await charge(order)
    try {
      await sendPackage(order)
    } catch (e) {
      await refund(order)
      throw e
    }
  } catch (e) {
    await unreserve(order)
    throw e
  }
}
```

```ts
// activities.ts

import { Context } from "@temporalio/activity"
import {
  FulfillmentResult,
  fulfillmentService,
  inventoryService,
  PaymentResult
  paymentService,
  ReservationResult
} from './services'
import { Order} from './types'

const getRequestId = () => Context.current().info.workflowExecution.workflowId

export async function reserve(order: Order): Promise<ReservationResult> {
  return inventoryService.reserve({...order, requestId: getRequestId()})
}
export async function unreserve(order: Order): Promise<ReservationResult> {
  return inventoryService.unreserve({...order, requestId: getRequestId()})
}

export async function charge(order: Order): Promise<ReservationResult> {
  return inventoryService.charge({...order, requestId: getRequestId()})
}
export async function refund(order: Order): Promise<ReservationResult> {
  return inventoryService.refund({...order, requestId: getRequestId()})
}

export async function sendPackage(order: Order): Promise<ReservationResult> {
  return inventoryService.sendPackage({...order, requestId: getRequestId()})
}
```

# Writing universal modules for Deno, Node and the browser

Luca Casonato showed us how easy it is to develop with Deno and transpile Deno to Node.

🔖 It's interesting to build a Deno module by following Luca's talk.

---

Recommend: ❤️❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/writing-universal-modules-for-deno-node-and-the-browser-700)

Repository: [Demo on GitHub](https://github.com/lucacasonato/greeter)

---

### Explore how easy it is to build things for / with Deno

Writing TypeScript with Deno is really boring since TypeScript works everywhere out of the box with zero configuration.

### Create a library that creates greeting messages

Install VSCode extension for Deno. Run initialize Deno command `Deno: Initialize Workspace Configuration`.

```ts
// mod.ts

export enum Greeting {
  Hello = "Hello",
  Hi = "Hi",
  GoodEvening = "Good evening",
}

export function greet(
  name: string,
  greeting: Greeting = Greeting.Hello
): string {
  return `${greeting} ${name}`
}
```

Start Deno with `$ deno`. Import our `Greeting` and `greet`. Then we can `greet("Leo", Greeting.Hi)`.

### Add unit tests using Deno's built-in test framework

Deno has a built-in test runner `$ deno test`. Very simple interface, but allows for advanced capabilities.

```ts
// mod_test.ts

import { greet, Greeting } from "./mod.ts"
import { assertEquals } from "https://deno.land/std@0.134.0/testing/asserts.ts"

Deno.test("greet default", () => {
  const greeting = greet("TypeScript Congress")
  assertEquals(greeting, "Hello TypeScript Congress!")
})

Deno.test("greet hi", () => {
  const greeting = greet("TypeScript Congress", Greeting.Hi)
  assertEquals(greeting, "Hi TypeScript Congress!")
})

Deno.test("greet good evening", () => {
  const greeting = greet("TypeScript Congress", Greeting.GoodEvening)
  assertEquals(greeting, "Good evening TypeScript Congress!")
})
```

### Format and lint the code using Deno's built-in tooling

- Deno has a built-in formatter for JS, TS, JSON and Markdown: `$ deno fmt`
  - Opinionated to ensure consistent style
  - Very similar style to prettier, but much faster
- Deno has a built-in linter for JS and TS: `$ deno lint`
  - Does not check formatting
  - Catches logic errors
  - Enforces styling

### Publishing for Deno first: deno.land/x

- Deno first nodule registry
- Not a "blessed" registry. You can host modules anywhere. Even on your own domain!
- Immutable (versions / nodules can not be deleted)
- Hooks into your existing GitHub workflow (literally)
- Learn more at https://deno.land/x

### View auto-generated docs using doc.deno.lang

- Deno provides a documentation generator OOTB
- Can be used in CLI: `$ deno doc`
- Also available as a website: https:doc.deno.land

TypeScript annotations & JSDoc comments are used to generate docs right from your code.

### Test the code in Node, and publish to NPM

- Deno supports loading .ts files natively, Node and the browser do not
  - We need to emit .js files for these
- Node consumes packages from NPM
  - We need to publish to NPM
- Node does not support all web APIs
  - You might need polyfills

Introducing: DNT

dnt = deno node transform

- Does transpilation to CJS and pure JS ESM for distribution on NPM
- Automatically replaces globals not available in Node with polyfills
- Transpiles tests, and runs them in Node

Best of both worlds:

- Develop using all of the built-in Deno tooling
- Still make modules available to users writing for Node

```ts
// _build.ts

import { build, emptyDir } from "https://deno.land/x/dnt/mod.ts"

await emptyDir("./npm")

await build({
  entryPoints: ["./mod.ts"],
  outDir: "./npm",
  shims: {
    deno: "dev",
  },
  package: {
    // package.json properties
    name: "@wtlin1228/greeter",
    version: Deno.args[0],
    description: "A demo project for TypeScript Congress talk",
    license: "MIT",
  },
})

Deno.copyFileSync("README.md", "npm/README.md")
```

Run `$ deno run -A _build.ts 0.0.1`. Then `$ npm publish` 🚀

# Plug-in architecture: how TypeScript let us paint-by-numbers

Lili Kastilio showed us how TypeScript could help us when we are adding a new feature to an unfamiliar codebase.

🔖 The DX of using patterns by leveraging TypeScript is great. TypeScript will hint you as long as the interfaces are defined.

---

Recommend: ❤️❤️❤️❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/plug-in-architecture-how-typescript-let-us-paint-by-numbers)

# How to properly handle URL slug changes in Next.js

Ondrej Polesny showed us how to handle dynamic routes with [`getStaticPaths`](https://nextjs.org/docs/basic-features/data-fetching/get-static-paths) and [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching/get-static-props) in `next.js`.

🔖 Personally, I would read the documentation of `next.js` to learn how to handle it.

---

Recommend: ❤️

Video: [Watch on GitNation](https://portal.gitnation.org/contents/how-to-properly-handle-url-slug-changes-in-nextjs)

Repository: [Demo on GitHub](https://github.com/ondrabus/kontent-boilerplate-next-js-ts-congress-2022)
