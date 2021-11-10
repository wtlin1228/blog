---
title: Extend Your GraphiQL
excerpt:
date: 2021-10-10
tags: [GraphQL, GraphiQL, Django, Graphene]
slug: extend-your-graphiql
cover: cover.jpg
---

This post will show you how to extend your `GraphiQL` with extensions, making it a tool that makes everyone happy.

## Context

I felt a little frustrated the first time I opened our team's `GraphiQL`. I couldn't see the whole picture of our schema. It's difficult to play around queries and mutations to understand our data structure.

Therefore, I decide to extend our `GraphiQL` with the extensions provided by [OneGraph](https://github.com/onegraph/graphiql-with-extensions) for me and also for my team.

## Technology Overview

Here is our technology stack:

- [Django](https://www.djangoproject.com/)
- [GraphQL](https://graphql.org/)
- [Graphene](https://graphene-python.org/)

## Steps

1. Copy `static.html` from [OneGraph/graphiql-with-extensions](https://github.com/OneGraph/graphiql-with-extensions/blob/master/examples/static.html).
2. Put `static.html` into `templates/graphene`.
3. Modify the file if you need to do additional work.
4. Get ready to play around your brand new `GraphiQL` and share it with your team!

<img src="https://user-images.githubusercontent.com/476818/51567716-c00dfa00-1e4c-11e9-88f7-6d78b244d534.gif" width="640" />

That's all. Thanks [OneGraph](https://github.com/OneGraph) and [Overriding templates](https://docs.djangoproject.com/en/3.2/howto/overriding-templates/) for making it so easy.
