---
title: DevOps in JunyiAcademy
excerpt: DevOps in JunyiAcademy contains version control, CI/CD and release management.
date: 2021-07-12
tags: [junyiacademy, devops]
slug: devops-in-junyiacademy
cover: cover.jpg
---

DevOps in JunyiAcademy contains version control, CI/CD and release management.

## Version Control

Follow both [SemVer](https://semver.org/) and [CalVer](https://calver.org/). Version will increase when merge `staging` into `master`. And the release manager will create tags for each version.

Convention: MAJOR.MINOR.PATCH(-rc.[x])-yyyy-mm-dd-prod[x]

- MAJOR increments if there is any breaking change
- MINOR increments if there is any new feature
- PATCH increments if there is any release application
- rc.[x] increments if there is no release application
- yyyy-mm-dd is based on the project's release calendar
- prod[x] indicate which production environment is deployed to

### Example

Current version is `1.3.10-rc.2-2020-05-17-prod1`.

When `staging` is merged into `master`, next version will be:

- `2.3.0-2020-05-20-prod2` if there is any breaking change
- `1.4.0-2020-05-20-prod2` if there is any new feature
- `1.3.11-2020-05-20-prod2` if there is any release application
- `1.3.10-rc.3-2020-05-20-prod2` if there is no release application

## CI / CD

Implement CI/CD with GitLab Pipelines and GitLab Runner.

The CI/CD pipeline will be triggered after:

- push a new commit (lint -> test -> deploy -> bvt)
- merge a branch (lint -> test -> deploy -> bvt)
- fire a scheduled job (ex: auto merge, version switch, daily test)
- fire a job manually

### Stage - Lint

- prettier - check the format of the frontend code
- eslint - find and fix problems in the frontend code
- yapf - check the format of the backend code
- pylint - find and fix problems in the backend code

### Stage - Test

- external api test
- backend api test
- backend unit test
- frontend unit test

### Stage - Deploy

- deploy to live server - connect to production database
- deploy to test server - connect to test database
- stop live server - will be triggered when branch is deleted or merged
- stop test server - will be triggered when branch is deleted or merged
- deploy backend live server - manually deploy backend with production database
- deploy backend test server - manually deploy backend with test database
- update changelog - only when a new tag is created

### Stage - BVT

- e2e test
- snapshot test

### Stage - Scheduled

- auto merge - merge staging into master and deploy to production server 1~3 at morning
- switch version - switch version after all contributors confirm their changes at noon
- external api test - test the external APIs in `master` branch twice a day
- e2e test - run cypress tests in `master` branch twice a day

## Release Management

Switch to the new version and update the changelog automatically if the newest created tag doesn't be marked as release candidate at noon. This operation can also be done by release manager manually.

### Changelog

Changelog is sorted and grouped by MR titles with the following information:

- live server url - the live server url will be preserved for two weeks (it's extremely useful for QA)
- merged requests - titles and authors of merged MRs

### Prevent Emergencies

In order to switch back to the stable version when emergencies happen in the current version, we keep the latest 3 released versions alive. That's why we put `prod[x]` in the version convention.
