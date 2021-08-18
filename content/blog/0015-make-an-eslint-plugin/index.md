---
title: Make an ESLint plugin for your team
excerpt: There must be more and more rules added to your team's style guide as the team grows up. And that's a good thing actually. But it's still hard to make everyone follow the style guide especially for those who just joined the team. As a frontend engineer, ESLint is just what we need.
date: 2021-08-18
tags: [eslint, AST, abstract-syntax-trees]
slug: make-an-eslint-plugin
cover: cover.jpg
---

There must be more and more rules added to your team's style guide as the team grows up. And that's a good thing actually. But it's still hard to make everyone follow the style guide especially for those who just joined the team. As a frontend engineer, ESLint is just what we need.

## Create an ESLint Plugin

The easiest way to start creating a plugin is to use the [Yeoman generator](https://www.npmjs.com/package/generator-eslint). The generator will guide you through setting up the skeleton of a plugin.

You can add your first rule with `yo eslint:rule` after creating a plugin. A rule includes three parts: source file, test file and documentation.

### Rule - Source File

[AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree)(Abstract Syntax Tree) and the [visitor pattern](https://en.wikipedia.org/wiki/Visitor_pattern) is essential when developing ESLint plugin (Babel and Codemods plugins also use the same tool).

Take the `limit-continuous-import-declarations` rule as example:

```js
module.exports = {
  create(context) {
    // use context to share these variables between callbacks
    // of each selectors while traversing the abstract syntax
    // tree.
    const limit = 10
    let continuousImportDeclarations = 0
    let lastEnd = 0

    return {
      ImportDeclaration(node) {
        const [start, end] = node.range
        if (start !== lastEnd + 1) {
          // reset counter if there is an empty line between
          // import declarations
          continuousImportDeclarations = 0
        }
        continuousImportDeclarations += 1
        lastEnd = end

        if (continuousImportDeclarations > limit) {
          context.report({
            node: node,
            message: "Too many continuous import declarations",
          })
        }
      },
    }
  },
}
```

Besides, make the rule more complete by providing a meta object containing `type`, `docs`, `fixable`, `schema`, ...etc.

Note: `Program:exit` is helpful when the rule needs to wait until the whole file is parsed.

### Rule - Test File

Remember to configure the correct parser for your `ruleTester`. For example, set the parser to `@babel/eslint-parser` if you want to use ES6+ features.

```js
import { RuleTester } from "eslint"

const ruleTester = new RuleTester({
  parser: require.resolve("@babel/eslint-parser"),
  parserOptions: {
    requireConfigFile: false,
  },
})
```

Here is part of the unit tests in the `limit-continuous-import-declarations` rule:

```js
import rule from "rules/limit-continuous-import-declarations"

const testCase = [
  "import foo1 from 'foo'",
  "import foo2 from 'foo'",
  "import foo3 from 'foo'",
].join("\n")

ruleTester.run("limit-continuous-import-declarations", rule, {
  valid: [
    {
      code: testCase,
      options: [{ limit: 5 }],
    },
  ],
  invalid: [
    {
      code: testCase,
      options: [{ limit: 2 }],
      errors: [
        {
          message: "Too many continuous import declarations",
        },
      ],
    },
  ],
})
```

## Use the ESLint Plugin

Use the plugin by configuring the ESLint configuration file like this:

```js
module.exports = {
  // ...

  plugins: ["plugin-name"],

  rules: {
    "plugin-name/your-rule": [
      "error",
      {
        // provide options if you have
      },
    ],
  },

  // ...
}
```

Or if the plugin provides some configs like `recommended`. Then the plugin can be used like this:

```js
module.exports = {
  // ...

  extends: ["plugin:plugin-name/recommended"],

  // ...
}
```

## Complete Example

The complete source code is in this github repository: https://github.com/wtlin1228/eslint-plugin-wtlin.
