---
title: 2022 in Review
excerpt: 2022-in-review
date: 2023-01-05
tags: [2023, review]
slug: 2022-in-review
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

## Tap Dance

In 2022, I started learning tap dance and I'm excited for my first performance in January 2023.

![tap dance](./images/tap-dance.jpg)

## Cloud Architecture

I completed three courses hosted by [AWS Training and Certification](https://aws.amazon.com/training/), which helped me become more confident in designing reliable cloud architecture.

![aws multi-tier architecture](./images/aws-multi-tier-architecture.jpg)

## Better Monorepo Management

To cache our task results and explore the dependency graph, I introduced [Nx](https://nx.dev/) into our monorepo. It was surprisingly easy to turn our repo into a Nx package-based repo and then further into a Nx integrated-based repo step by step.

## Better Testing Strategy

![testing matrix](./images/testing-matrix.png)

I convinced my team to implement the following actions to optimize for maximum confidence at minimum effort:

- use [testing-library](https://testing-library.com/) for component tests instead of [enzyme](https://enzymejs.github.io/enzyme/)
- use [msw](https://mswjs.io/) for integration tests
- perform [interaction tests](https://storybook.js.org/docs/react/writing-tests/interaction-testing) in our UI library's Storybook.

## Better Component API Interface

When designing the new component API interface, I aimed for a small API surface area that ensures consistency. I had a [note](https://github.com/wtlin1228/dev-note/blob/main/docs/design-system-hack-to-create-good-interface.md) for myself.

## My Blog

Last year, I published ten blog posts:

1. [Summary of TestJS Summit 2021](https://leonerd.gatsbyjs.io/0020-test-js-summit-2021/)
1. [Handle API request race conditions in React](https://leonerd.gatsbyjs.io/0021-api-request-race-conditions/)
1. [Visualize fiber tree with ASCII Art](https://leonerd.gatsbyjs.io/0022-visualize-fiber-tree-with-ascii-art/)
1. [Test the shared UI Library built with Storybook](https://leonerd.gatsbyjs.io/0023-test-the-shared-ui-library-built-with-storybook/)
1. [Summary of TypeScript Congress 2022](https://leonerd.gatsbyjs.io/0024-typescript-congress-2022/)
1. [React at Scale with Nx](https://leonerd.gatsbyjs.io/0025-react-at-scale-with-nx/)
1. [Baby Rustacean - Ownership](https://leonerd.gatsbyjs.io/0026-baby-rustacean-ownership/)
1. [Design System](https://leonerd.gatsbyjs.io/0028-design-system/)
1. [Deduplicate JS Bundles](https://leonerd.gatsbyjs.io/0029-deduplicate-js-bundles/)
1. [Empower Testing Library with Custom Query](https://leonerd.gatsbyjs.io/0030-empower-testing-library-with-custom-query/)

## Chrome Extension

I created a chrome extension called [Web Presenter](https://chrome.google.com/webstore/detail/web-presenter/fcelpdljejcagbhalelapbihcccjkefn) that enables me to present directly on any website without preparing slides. It's also useful when I want to capture UI screenshots and attach them to my PRs.

## Open Source

Last year, I contributed to several open-source projects such as `facebook/react`, `mdn/interactive-examples`, `rust-lang/book`, `testing-library/testing-library-docs` and `remix-run/remix`, among others.

## The Future

Looking ahead to 2023, I have several exciting goals I hope to achieve:

- Launch my new website
- Encourage my team to adopt the [Fragment Masking Pattern](https://the-guild.dev/blog/unleash-the-power-of-fragments-with-graphql-codegen) to create more isolated and reusable UI components with GraphQL
- Transition our error handling interface to the [GraphQL Error Handling Interface](https://the-guild.dev/blog/graphql-error-handling-with-fp#better-modelisation-of-errors-using-union-and-interfaces) from the standard GraphQL "errors"
- Migrate our monorepo to an Nx integrated-based repo from an Nx package-based repo (https://nx.dev/concepts/integrated-vs-package-based)

In addition, I'm preparing for my relocation to Japan 🇯🇵.
