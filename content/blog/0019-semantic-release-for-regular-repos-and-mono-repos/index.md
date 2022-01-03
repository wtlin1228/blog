---
title: Semantic Release for Regular Repos and Monorepos
excerpt:
date: 2021-12-12
tags: [devops, semantic-release, lerna, github-actions]
slug: semantic-release-for-regular-repos-and-monorepos
cover: cover.jpg
---

> `semantic-release` automates the whole package release workflow including: determining the next version number, generating the release notes, and publishing the package.

In my opinion, `semantic-release` not only improves our development experience but also encourages us to write better commit messages. Therefore, I want to make a note of how I use it in both regular repos and monorepos with `lerna`.

## Overview

### Regular Repository

1. New commits pushed to `master` branch
2. Trigger GitHub Action - `Release`
3. Bump to version `x.y.z` based on those new commits
4. Update `CHANGELOG` for version `x.y.z`
5. Create `tag`, `release` for version `x.y.z`
6. Publish version `x.y.z` to the registry

### Monorepo Managed by Lerna

1. New commits pushed to `master` branch
2. Trigger GitHub Action - `Release`
3. Bump to version `x.y.z` based on those new commits
4. Update `CHANGELOG` for root package and sub-packages
5. (optional) Update version to `x.y.z` for root package
6. (optional) Update versions of `peerDependencies` for sub-packages
7. Create `tag`, `release` for version `x.y.z`
8. Merge `master` branch back to `develop` branch
9. Publish version `x.y.z` to the registry

## Setup

The following examples will use `GitHub Actions` as CI tool.

### Regular Repository

It's easy to use `semantic-release` in regular repositories. The automation can simply be set up by only two files.

1. Configure `semantic-release` by creating `.releaserc`

   ```json
   {
     "branches": ["master"],
     "plugins": [
       "@semantic-release/commit-analyzer",
       "@semantic-release/release-notes-generator",
       "@semantic-release/github",
       "@semantic-release/changelog",
       "@semantic-release/npm",
       {
         "path": "@semantic-release/git",
         "assets": ["CHANGELOG.md", "package.json"],
         "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
       }
     ]
   }
   ```

   `semantic-release` uses the Plugin Pattern. That's why we need to configure the plugins in `.releaserc`. See the [official document](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/plugins.md#plugins) for more information.

2. Create release workflow by creating `.github/workflows/release.yml`

   ```yml
   name: Release
   on:
     push:
       branches:
         - master
   jobs:
     release:
       name: Release
       runs-on: ubuntu-latest
       steps:
         - name: Checkout
           uses: actions/checkout@v2
           with:
             fetch-depth: 0

         - name: Setup Node.js
           uses: actions/setup-node@v2
           with:
             node-version: "16"

         - name: Install dependencies
           run: npm install

         - name: Build static assets
           run: npm run build

         - name: Install dependencies for semantic-release
           run: npm install -g semantic-release @semantic-release/commit-analyzer @semantic-release/release-notes-generator @semantic-release/github @semantic-release/changelog @semantic-release/npm @semantic-release/git

         - name: Release
           env:
             GITHUB_TOKEN: ${{ secrets.GITHUB_PERSONAL_ACCESS_TOKEN }}
             NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
           run: npx semantic-release
   ```

   Because I won't release and publish manually. I install `semantic-release` and it's dependencies here instead of listing them in the `package.json`.

   Note that `GITHUB_TOKEN` and `NPM_TOKEN` must be set properly.

### Monorepo Managed by Lerna

Lerna is a tool for managing JavaScript projects with multiple packages. Usually, we rely on `lerna version` and `lerna publish` in our release workflow.

1.  Configure `lerna.json`

    ```json
    {
      // ...
      "command": {
        "publish": {
          "message": "chore(release): %s",
          "registry": "https://registry.npmjs.org"
        }
      },
      "changelogPreset": {
        "name": "conventionalcommits"
      }
    }
    ```

    - `command.publish.message` customizes commit messages for release commits.
    - `command.publish.registry` is where to publish those packages to.
    - `changelogPreset.name` replaces the default `Angular preset`. See [this discussion](https://github.com/lerna/lerna/issues/2668#issuecomment-915774425) for more information.

2.  Add `release` and `publish` scripts in `package.json`

    ```json
    {
      "scripts": {
        // ...
        "release": "lerna version --yes --conventional-commits --create-release github",
        "publish": "lerna publish --yes from-git"
      }
    }
    ```

3.  Create release workflow by creating `.github/workflows/release.yml`

    ```yml
    name: Release
    on:
      push:
        branches:
          - master
    jobs:
      release:
        name: Release
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v2
            with:
              fetch-depth: 0

          - name: Setup Node.js
            uses: actions/setup-node@v2
            with:
              node-version: "16"

          - name: Install dependencies
            run: npm install

          - name: Build static assets
            run: npm run build

          - name: Authenticate with Registry
            run: echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > ~/.npmrc
            env:
              NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

          - name: Configure CI Git User
            run: |
              git config --global user.email some-email@foo.com
              git config --global user.name GitHub Actions

          - name: Release
            env:
              GH_TOKEN: ${{ secrets.GITHUB_PERSONAL_ACCESS_TOKEN }}
            run: npm run release

          - name: Merge Master back to Develop
            run: |
              git fetch
              git checkout develop
              git pull
              git merge master
              git push
              git checkout master

          - name: Publish
            run: npm run publish
    ```

    `--conventional-commits` flag in `lerna version` is the key to use `conventional-changelog` to determine version bump and generate CHANGELOG.

Note that `GITHUB_TOKEN` and `NPM_TOKEN` must be set properly.

3.  Update version for root package and peer dependencies for sub-packages (optional)

    Add a life cycle script `version` in `package.json`

    ```json
    {
      "scripts": {
        // ...
        "version": "zx ./scripts/version.mjs"
      }
    }
    ```

    Use `zx` to write our script for `version`.

    ```js
    // ./scripts/version.mjs

    #!/usr/bin/env zx

    //------------------------------------------------------------------------------
    // Helpers
    //------------------------------------------------------------------------------
    const getMajor = version => version.split(".")[0]

    //------------------------------------------------------------------------------
    // Update version of root-package
    //------------------------------------------------------------------------------
    const { version: currentVersion } = require("../package.json")
    const { version: nextVersion } = require("../lerna.json")

    if (currentVersion !== nextVersion) {
      await $`npm version ${nextVersion} --no-git-tag-version`
    }

    //------------------------------------------------------------------------------
    // Update peerDependencies of sub-packages only if bump major
    //------------------------------------------------------------------------------
    if (getMajor(currentVersion) !== getMajor(nextVersion)) {
      const { fs } = require("zx")

      const dirList = await fs.readdir("./packages")

      for (const dir of dirList) {
        const subPackageJSON = require(`../packages/${dir}/package.json`)

        if (!subPackageJSON.devDependencies) {
          continue
        }

        Object.entries(subPackageJSON.devDependencies).forEach(
          ([packageName, packageVersion]) => {
            if (packageName.startsWith("@your-preset/")) {
              subPackageJSON.peerDependencies[packageName] = packageVersion
            }
          }
        )

        fs.writeFileSync(
          `./packages/${dir}/package.json`,
          JSON.stringify(subPackageJSON, null, 2) + "\n"
        )
      }
    }

    //------------------------------------------------------------------------------
    // Stage all changes
    //------------------------------------------------------------------------------
    await $`git add .`
    ```

## Reference

- https://github.com/semantic-release/semantic-release
- https://github.com/lerna/lerna
- https://docs.github.com/en/actions
- https://github.com/google/zx
