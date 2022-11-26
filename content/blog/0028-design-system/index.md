---
title: Design System
excerpt: design-system
date: 2022-10-28
tags: [design-system]
slug: design-system
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

# Outline

```
What makes a good component library?
├── What is a good component API?
│   ├── Consist
│   └── Flexible
├── Tiktok's Approach
├── Shopify's Approach
├── Google's Approach
└── GitHub's Approach
How to quantify a component library's effectiveness?
├── Adoption Rate
│   └── Component Coverage
├── More Metrics
└── How to collect metrics?
How to handle the breaking changes?
Reference
```

A lot of people have a big misconception that design systems mean you just isolate things at their component level. And just design the button, and just design the headings, and just design the card in isolation. But that's certain not true. It's important to realize that things do come together and form a cohesive page at the end of the day.

Another misconception is that, when we talk about design systems, we just want everything to look the same everywhere. That's really not true, because, if your metrics are doing well and the buttons are different on the checkout page than the product detail page, then that's fine.

And design system is worthless if not used. Design system is a tool. It's not enough to exist. It has to actively be used to make the product better or helper deliver it faster. That's the only way for a design system to be valuable. Building a design system without getting sufficient adoption is a waste of time and effort.

We want to create a [design system] that results in:

- Avoid inconsistent naming across the origination.
- Short development times and consistent code quality.
- A beautiful, visually consistent experience.

# What makes a good component library?

Good component API.

## What is a good component API?

It depends! And there is a spectrum of flexibility.

`Consistent <--------------> Flexible`

### Consistent

- Easy to use
- Consistent output
- Easy to implement

### Flexible

- Covers more use-cases

## Tiktok's Approach

- We don't need to/cant's cover 100% of use-cases
- Aim for 90%, and try to make the remaining 10% easy to do yourself
- Therefore: Start near the left with a rigid API that focuses on solving the 'hard' problems:
  - Accessibility features
  - Animation
- Move further right if necessary (sometimes can involve breaking changes)
- Be careful; moving further right means we reduce consistency, increase complexity

## Shopify's Approach

- Polaris
- Opinionated
- Strict API
- Predictable and consistent outcomes
- Can become rigid and restricting

## Google's Approach

- Material UI
- Flexible API
- Extremely flexible and context dependent
- Can become messy and fragmented

## GitHub's Approach

- Primer
- Sit in the middle of the Spectrum of flexibility
- Use composition to allow flexibility without ejecting
- Design System are for People

# How to quantify a component library's effectiveness?

Key metric: adoption rate

## Adoption Rate

How much engineer wants to use your components?

For example, you have a email field. If it isn't in the right format, it would have a red border, an error message below it. That's say you don't have that. Then the developers would build themselves, which is going to be a huge waste of develop time.

### Component Coverage

Component coverage value for a given `Button` component is equal to

`(# of good cases) / (# of good cases + # of bad cases)`

```jsx
// Good Cases
<Button />

// Bad Cases
<MyOwnButton />
<a className="but-primary" >
```

Let's say that `Button` has 3 good cases and 9 bad cases. Thus, `Button` has a component coverage of 3 / (3 + 9) = 0.25 = 25%

## More Metrics

- Repo adoption rate
- Version distribution
- Styling standards adoption rate
- Bug fixes
- Feature requests
- Component library related linting rule violations
- etc.

## How to collect metrics?

You could use AST to do parsing and traversing your source code.

Another choice would be [radius tracker](https://github.com/rangle/radius-tracker) which is a fully automated tool for collecting design system stats from the codebase.

# How to handle the breaking changes?

- Don't leave consumers behind
- use a monorepo like `Nx`
- Follow the one version rule
- Code ownership: When we make a breaking change in our package, it's our responsibility to upgrade our consumers - not the consumers'

# Reference

- [Find Out If Your Design System Is Better Than Nothing – Arseny Smoogly, React Summit 2022](https://portal.gitnation.org/contents/find-out-if-your-design-system-is-better-than-nothing)
- [Walking the Line Between Flexibility and Consistency in Component Libraries](https://portal.gitnation.org/contents/walking-the-line-between-flexibility-and-consistency-in-component-libraries)
- [Developing and Driving Adoption of Component Libraries](https://portal.gitnation.org/contents/developing-and-driving-adoption-of-component-libraries)
- [Design Systems with Brad Frost - The State of the Web](https://youtu.be/2M6dJ2Uynhg)
- [Atomic Design by Brad Frost](https://atomicdesign.bradfrost.com/)
- [Website Style Guide Resources](http://styleguides.io/)
- [design.systems](https://design.systems/)
