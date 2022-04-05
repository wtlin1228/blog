---
title: Visualize fiber tree with ASCII Art
excerpt:
date: 2022-04-05
tags: [react, deep-dive]
slug: visualize-fiber-tree-with-ascii-art
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

- #1: Visualize fiber tree with ASCII Art
- #2: ...

---

This series of blog posts is about studying the source code of [React 18](https://github.com/facebook/react/tree/v18.0.0). In this post, I want to show how I visualize the fiber tree with ASCII Art. So I can inspect the `workInProgress` tree and `current` tree with my tools as below:

```
HostRoot─────App───────A─────div──────B1─────div──────C1────'c1'
                                       │               │
                                       │              C2────'c2'
                                       │               │
                                       │              C3────'c3'
                                       │
                                      B2─────div──────C4────'c4'
                                       │               │
                                       │              C5────'c5'
                                       │               │
                                       │              C6────'c6'
                                       │
                                      B3─────div──────C7────'c7'
                                                       │
                                                      C8────'c8'
                                                       │
                                                      C9────'c9'
```

<img src="https://media.giphy.com/media/8lvpUQmGJXaP8JjUcx/giphy.gif" width="100%" >

# Determine how to display fibers

I need to decide what to display for a fiber. Because each fiber has a tag. I can determine the display name based on it. In this case, I only care about these five tags:

- `FunctionComponent`
- `IndeterminateComponent`: Before we know whether it is function or class.
- `HostRoot`: Root of a host tree. Could be nested inside another node.
- `HostComponent`: Could be `div`, `p`, ... in DOM.
- `HostText`

```ts
// My debug tools.
// packages/react-reconciler/leonerdDebugTools.js

import type { Fiber } from "./src/ReactInternalTypes"

import {
  FunctionComponent,
  IndeterminateComponent,
  HostRoot,
  HostComponent,
  HostText,
} from "./src/ReactWorkTags"

function getFiberDisplayName(fiber: Fiber | null): string {
  if (fiber === null) {
    return ""
  }

  switch (fiber.tag) {
    case FunctionComponent:
    case IndeterminateComponent:
      return fiber.elementType.displayName

    case HostRoot:
      return "HostRoot"

    case HostComponent:
      return fiber.elementType

    case HostText:
      return fiber.stateNode?.textContent || fiber.pendingProps
  }
}
```

# Generate the ASCII Art fiber tree

## Each fiber should only be drawn once

I build a lookup map with [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) because a Map's keys can be any value (including functions, objects, or any primitive). In this case, the keys are the fiber objects.

```ts
const drawnFibersMap = new Map<Fiber, boolean>()

// add fiber to the lookup map
drawnFibersMap.set(fiber, true)

// check is fiber in the lookup map
drawnFibersMap.get(fiber)
```

## General pattern of React

I would use this general pattern of React to traverse the fiber tree.

```ts
function doSomething(fiber) {
  // do stuff depending on fiber.tag

  if (fiber.child) {
    doSomething(fiber.child)
  }

  if (fiber.sibling) {
    doSomething(fiber.sibling)
  }
}
```

**Note**: fibers use the pointers: `return`, `sibling` and `child` to connect with other fibers.

## Draw the ASCII Art lines if fiber is the leaf node

I need to keep the traversing path since I only draw the ASCII Art lines when `fiber.child` is `null`. Extend the general pattern:

```ts
function gatherAsciiArtTreeRecursively(fiber, path) {
  if (fiber.child === null) {
    // generate the ASCII Art lines here
  }

  if (fiber.child) {
    gatherAsciiArtTreeRecursively(fiber.child, [...path, node.child])
  }

  if (fiber.sibling) {
    gatherAsciiArtTreeRecursively(fiber.sibling, [
      ...path.slice(0, path.length - 1),
      fiber.sibling,
    ])
  }
}
```

## Color fibers with `Chalk`

I would use `Chalk` to do the color staff since it has been installed inside React already.

```ts
// My debug tools.
// packages/react-reconciler/leonerdDebugTools.js

import chalk from "chalk"

const CORAL_CHALK = "FE7D6A"
const PARAKEET_CHALK = "03C04A"

function makePadStartWithChalkColor(chalkColor: CORAL_CHALK | PARAKEET_CHALK) {
  return function padStartWithChalkColor({
    source,
    padStartLength,
    padStartString,
  }: {
    source: string
    padStartLength: number
    padStartString: string
  }) {
    return (
      Array.from(
        { length: padStartLength - source.length },
        () => padStartString
      ).join("") + chalk.hex(chalkColor)(source)
    )
  }
}

const padStartWithCoralChalk = makePadStartWithChalkColor(CORAL_CHALK)
const padStartWithParakeetChalk = makePadStartWithChalkColor(PARAKEET_CHALK)
```

## Generate ASCII Art lines based on the path

`getAsciiArtLines` will be called when `fiber.child` is `null`. It will generate two ASCII Art lines:

- the first line: represent the current path
- the second line: act as a connector between the lines of current path and next path

```ts
// My debug tools.
// packages/react-reconciler/leonerdDebugTools.js

const PAD_START_LENGTH = 8

function getAsciiArtLines({
  path,
  lookupMap,
  workInProgress,
}: {
  path: Fiber[]
  lookupMap: Map<Fiber, boolean>
  workInProgress: Fiber | null
}) {
  let isFirstDashedPrint = true

  return [
    // first line, example:
    // "HostRoot─────App───────A─────div──────B1─────div──────C1────'c1'"
    path.reduce((acc, curr) => {
      if (lookupMap.get(curr)) {
        if (curr.sibling) {
          return acc + "│".padStart(PAD_START_LENGTH, " ")
        } else {
          return acc + " ".padStart(PAD_START_LENGTH, " ")
        }
      } else {
        lookupMap.set(curr, true)

        let padStartString = "─"
        if (isFirstDashedPrint) {
          isFirstDashedPrint = false
          padStartString = " "
        }

        const displayName = getFiberDisplayName(curr)

        if (curr === workInProgress) {
          return (
            acc +
            padStartWithCoralChalk({
              source: displayName,
              padStartLength: PAD_START_LENGTH,
              padStartString: padStartString,
            })
          )
        } else {
          return acc + displayName.padStart(PAD_START_LENGTH, padStartString)
        }
      }
    }, ""),
    // second line, example:
    // "                                       │               │        "
    path.reduce((acc, curr) => {
      const char = curr.sibling === null ? " " : "│"
      return acc + char.padStart(PAD_START_LENGTH, " ")
    }, ""),
  ]
}
```

**Note**: I pass `workInProgress` to `getAsciiArtLines` so I can color the `workInProgress` fiber with carol chalk.

**Note**: Each fiber could be colored with different chalk based on `fiber.flags` if I want to see which fiber is incomplete or something else.

# Hold the `FiberRoot`

It would be impossible to access the whole fiber tree if I weren't holding the `FiberRoot`. So I need to find an entry point then keep the `FiberRoot`. I choose to do it inside [updateContainer](https://github.com/facebook/react/blob/v18.0.0/packages/react-reconciler/src/ReactFiberReconciler.new.js#L381) of `ReactFiberReconciler.new.js`.

```ts
// packages/react-reconciler/src/ReactFiberReconciler.new.js

export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function
): Lane {
  // emit...

  const root = scheduleUpdateOnFiber(current, lane, eventTime)

  // I prefix my utils with "leonerd__" to distinguish them from the original code.
  leonerd__setFiberRoot(root)

  // emit...
}
```

```ts
// My debug tools.
// packages/react-reconciler/leonerdDebugTools.js

import type { FiberRoot } from "./src/ReactInternalTypes"

let fiberRoot

export function leonerd__setFiberRoot(root: FiberRoot): void {
  fiberRoot = root
}
```

# Draw the trees

## Draw the `current` tree

The entry point of `current` tree is `fiberRoot.current`.

<!-- prettier-ignore-start -->

```ts
// My debug tools.
// packages/react-reconciler/leonerdDebugTools.js

export function leonerd__showCurrentFiberTree(
  workInProgress: Fiber | null
): void {
  const asciiArtTree = getAsciiArtTree(
    fiberRoot.current, 
    workInProgress
  )

  console.log(
    [
      "current fiber tree:", 
      "\n", 
      ...asciiArtTree
    ].join("\n")
  )
}
```

<!-- prettier-ignore-end -->

## Draw the `workInProgress` tree

The entry point of `workInProgress` tree is `fiberRoot.current.alternate`.

<!-- prettier-ignore-start -->

```ts
// My debug tools.
// packages/react-reconciler/leonerdDebugTools.js

export function leonerd__showWorkInProgressFiberTree(
  workInProgress: Fiber | null
): void {
  const asciiArtTree = getAsciiArtTree(
    fiberRoot.current.alternate,
    workInProgress
  )

  console.log(
    [
      "workInProgress fiber tree:", 
      "\n", 
      ...asciiArtTree
    ].join("\n")
  )
}
```

<!-- prettier-ignore-end -->
