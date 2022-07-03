---
title: React at Scale with Nx
excerpt: Use Nx to scale your React project
date: 2022-06-26
tags: [nx, monorepo, scale]
slug: nx-monorepo
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

# Overview

## Why not Lerna?

Lerna is more like a multi-repo approach. But they just happen to be in the same git repo.

- [Lerna is dead — Long Live Lerna](https://blog.nrwl.io/lerna-is-dead-long-live-lerna-61259f97dbd9)
- [From Lerna/Yarn to Nx: Faster Build Times and Better Dev Ergonomics](https://blog.nrwl.io/lerna-yarn-nx-faster-build-times-better-dev-ergonomics-2ec28463d3a5)
- [Migrating from Lerna to Nx: Better Dev Ergonomics + Much Faster Build Times](https://blog.nrwl.io/migrating-from-lerna-to-nx-better-dev-ergonomics-much-faster-build-times-da76ff14ccbb)

## Why monorepos?

- Atomic changes: shorten the change cycle time
- Shared code: don't need to publish your code to share it
- Single set of dependencies: update once and keep everything up-to-date

## Why not code collocation?

- Running unnecessary tests: only test the affected code
- No code boundaries: don't want to share all the code
- Inconsistent tooling

## Nx can help:

### Faster command execution

- Executors
- nx affected
- Local and distributed caching

### Controlled code sharing

- Library API
- Tags
- Publishable libraries
- CODEOWNERS

### Consistent coding practices

- Linting
- Generators
- Nrwl plugins
- Community plugins

### Accurate architecture diagram

- nx dep-graph

# Workspace Folder Structure

```
myorg/
├── apps/
├── libs/
├── tools/
├── workspace.json
├── nx.json
├── package.json
└── tsconfig.base.json
```

`/apps/` contains the application projects. This is the main entry point for a runnable application. Keeping applications as light-weight as possible, with all the heavy lifting being done by libraries that are imported by each application is recommended.

`/libs/` contains the library projects. There are many kinds of libraries, and each library defines its own external API so that boundaries between libraries remain clear.

`/tools/` contains scripts that act on your code base. This could be database scripts, custom executors, or workspace generators.

`/workspace.json` lists every project in your workspace. (this file is optional)

`/nx.json` configures the Nx CLI itself. It tells Nx what needs to be cached, how to run tasks etc.

`/tsconfig.base.json` sets up the global TypeScript settings and creates aliases for each library to aid when creating TS/JS imports.

# VSCode Extension

Nx provides a super useful VSCode extension [Nx Console](https://marketplace.visualstudio.com/items?itemName=nrwl.angular-console). I strongly recommend you to install it before getting into the Nx world.

# Create a New Workspace

`$ npx create-nx-workspace [workspace name]`

Workspace name sets three things:

- Directory (`/Users/.../my-org`)
- Path alias (`import {} from '@my-org/some-projects';`)
- npm scope (`npm install @my-org/published-library`)

ref: https://nx.dev/cli/create-nx-workspace

# Generators

Runs a generator that creates and/or modifies files based on a generator from a collection.

`$ nx generate [plugin]:[generator-name] [options]`

There are three main types of generators:

### Plugin Generators

Plugin Generators are available when an Nx plugin has been installed in your workspace.

For example, create an react app with `@nrwl/react` plugin generator.

`$ npm install -D @nrwl/react`

`$ npx nx generate @nrwl/react:app store`

### Workspace Generators

Workspace Generators are generators that you can create for your own workspace. [Workspace generators](https://nx.dev/generators/workspace-generators) allow you to codify the processes that are unique to your own organization.

For example, create a snippet for your organization.

`$ npx nx generate @nrwl/workspace:workspace-generator react-component-generator`

ref: https://nx.dev/generators/workspace-generators

### Update Generators

Update Generators are invoked by Nx plugins when you update Nx to keep your config files in sync with the latest versions of third party tools.

For example: migrate react to 18.

# Executors

Runs an Architect target with an optional custom builder configuration defined in your project.

`$ nx run <target> [options]`

For example, to build the `store` project, you can:

- `$ npx nx run store:build`
- or `$ npx nx build store` in shorthand
- or open `Nx Console` -> click `build` -> enter `store`

ref: https://nx.dev/cli/run

### Create custom targets

You can run any custom commands with Nx by creating new targets with `@nrwl/workspace:run-commands`.

ref: https://nx.dev/packages/workspace/executors/run-commands

# Libraries

There are many different types of libraries in a workspace. The four types of libraries below are recommended.

ref: https://nx.dev/structure/library-types#library-types

Note: Moving a library with Nx is easy with the [@nrwl/workspace:move](https://nx.dev/packages/workspace/generators/move) generator.

### Feature libraries

Developers should consider feature libraries as libraries that implement smart UI (with access to data sources) for specific business use cases or pages in an application.

For example, to generate a `feature-game-detail` feature library in the `libs/store` folder.

`$ npx nx g @nrwl/react:lib feature-game-detail --directory=store --appProject=store`

### UI libraries

A UI library contains only presentational components (also called "dumb" components).

For example, to generate a `ui-shared` react library in the `libs/store` folder.

`$ npx nx g @nrwl/react:lib ui-shared --directory=store`

Then generate a `header` component for `store/ui-shared`.

`$ npx nx generate @nrwl/react:component header --project=store-ui-shared --export`

Furthermore, configure the storybook for `store/ui-shared` lib.

`$ npx nx generate @nrwl/react:storybook-configuration store-ui-shared`

Nx will generate a cypress project `store-ui-shared-e2e` due to the previous command by default. So you can run the e2e test.

`$ npx nx e2e store-ui-shared-e2e`

### Data-access libraries

A data-access library contains code for interacting with a back-end system. It also includes all the code related to state management.

For example, data-access-authentication

### Utility libraries

A utility library contains low-level utilities used by many libraries and applications.

For example, to generate a `util-formatters` utility library in the `libs/store` folder.

`$ npx nx g @nrwl/workspace:library util-formatters --directory=store`

# Tags

Nx comes with a generic mechanism for expressing constraints: tags. You can use tags to impose constraints on the project graph.

Projects(apps and libs) can have multiple tags. So you can define the tags in multiple dimensions.

- scope dimension: `scope:store`, `scope:api`, `scope:shared`
- type dimension: `type:app`, `type:e2e`, `type:feature`, `type:ui`, `type:util`

Then you can apply your rules by setting the `depConstraints` in the root `.eslintrc.json`.

```json
"depConstraints": [
    {
        "sourceTag": "scope:store",
        "onlyDependOnLibsWithTags": ["scope:store", "scope:shared"]
    },
    {
        "sourceTag": "scope:api",
        "onlyDependOnLibsWithTags": ["scope:api", "scope:shared"]
    },
    {
        "sourceTag": "type:feature",
        "onlyDependOnLibsWithTags": ["type:feature", "type:ui", "type:util"]
    },
    {
        "sourceTag": "type:ui",
        "onlyDependOnLibsWithTags": ["type:ui", "type:util"]
    },
    {
        "sourceTag": "type:util",
        "onlyDependOnLibsWithTags": ["type:util"]
    }
]
```

ref: https://nx.dev/structure/monorepo-tags

# Example Repository

I have created an example repository https://github.com/wtlin1228/react-at-scale-with-nx. You can also follow the labs in this [workshop](https://github.com/nrwl/nx-react-workshop). It's good but is a little outdated. There is also a free course in [egghead.io](https://egghead.io/courses/scale-react-development-with-nx-4038?review=63954) if you prefer to watch videos.
