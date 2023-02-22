---
title: Building a React Component Library - Part 1
excerpt: building-a-react-component-library-part1
date: 2023-02-22
tags: [2023, ui-library, color-mode]
slug: building-a-react-component-library-part1
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

When talking about React component libraries, there are already plenty of good ones for us to choose from directly, like `material-ui`, `react-spectrum`, `fluentui`, `polaris`, etc. However, I just can't stop thinking about building one for myself to use in my upcoming site. So I list how and why those technical decisions are made for `wtlin-ui`.

- choose [Emotion][emotion] as the style engine
- add theme-based [system CSS props][system-css-properties] and [`sx` Prop][the-sx-prop] with [Styled System][styled-system]
- enable [Color Mode][prefers-color-scheme] without a flash of unstyled content by providing theme colors in the [CSS custom properties][css-custom-properties]

## Choose Emotion as the style engine

The trend of styling frontend applications is:

1. CSS
1. SASS
1. CSS-in-JS
1. Zero runtime

Som I was choosing from two CSS-in-JS libraries `styled-component`, `emotion` and two zero-runtime libraries `linaria`, `vanilla-extract`.

### Styled Component

For sure `styled-component` is one of the most popular and mature solutions, with good documentation. But the bundle size is larger than `emotion` and `emotion` also has better performance and flexibility.

See also:

- [Storybook Team - How we migrated 541 components from Styled Components to Emotion][storybook-styled-component]
- [MUI Team - We strongly recommend using Emotion for SSR projects][mui-styled-component]

### Linaria

Linaria is a zero-runtime CSS in JS library. CSS is extracted to CSS files during build time. So there is no JS bundle or runtime CPU overhead. Also, this brings caching benefits since these static CSS files may change at a different cadence than the JS files and also opens the door to the possibility of deduplicating styles (i.e. [Atomic CSS][atomic-css]).

Although all the features look very promising to me, the documentation is not top-notch, there isn't a dedicated website, no search feature and it feels like trial & error when trying to find a piece of information.

See also:

- [Airbnb Team - Airbnb’s Trip to Linaria][airbnb-trip-to-linaria]

### Vanilla Extract

Modern solution with great TypeScript integration and no runtime overhead. It's pretty minimal in its features, straightforward and opinionated. Everything is processed at compile time, and it generates static CSS files. Successor of Treat, also called "Treat v3", is developed and maintained by the same authors.

Code co-location is the main reason that stops me from choosing `vanilla-extract`.

## Add theme-based system CSS props and `sx` prop with Styled System

System CSS properties and the `sx` prop are getting more and more popular now. Many frameworks like `material-ui` support them nowadays. And I go for it since it helps simplify the components and prevent developers from jumping back and forth just to check the styles used only for doing some trivial layout.

See more:

- [MUI Team - The sx prop][mui-the-sx-prop]
- [GitHub Team - The sx prop][github-the-sx-prop]
- [Theme UI - The sx prop][theme-ui-the-sx-prop]

I don't like to write such code like this whenever I want to retrieve values from the theme.

<!-- prettier-ignore-start -->
```tsx
const Foo = () => {
  const theme = useTheme()
  return (
    <Box 
      mt={theme.sizing * 2}
    >
      Foo
    </Box>
  )
}
```
<!-- prettier-ignore-end -->

### The theme-based Props

Style functions of `styled-system` will try to find a value from the theme object, these could be deeply nested values, and fallback to a hard-coded value if they are unable to.

```ts
// font-size: 24px (theme.fontSizes[4])
<Box fontSize={4} />

// margin: 16px (theme.space[3])
<Box m={2} />

// color: #333 (theme.colors.blacks[0])
<Box color="blacks.3" />

// background color (theme.colors['light-red'])
<Box bg="light-red" />

// line-height: 1.5 (theme.lineHeights.copy)
<Box lineHeight="copy" />

// renders CSS `50%` width since it's not defined in theme
<Box width={1/2} />
```

### The `sx` Prop

Define the type `SxProp` and the style function `sx`:

```ts
// sx.ts
import css, { SystemStyleObject } from "@styled-system/css"

export interface SxProp {
  sx?: SystemStyleObject
}

const sx = (props: SxProp) => css(props.sx)

export default sx
```

Next step, combine it with `styled` API provided by `emotion` (or `styled-component`):

```ts
import styled from "@emotion/styled"
import sx, { SxProp } from "./sx"

type StyledBoxProps = SxProp
const Box = styled.div<StyledBoxProps>(sx)
```

Then, the `Box` component can accept `sx` prop and can be used like this:

```tsx
<Box
  sx={{
    mt: "2",
    p: "1",
    bg: "scale.red.3",
    color: "common.white",
  }}
>
  the sx prop works!
</Box>
```

The `sx` prop provides a lot of power, which means it is an easy tool to abuse. To best make use of it, we recommend following these guidelines:

- Use the `sx` prop for small stylistic changes to components. For more substantial changes, consider abstracting your style changes into your own wrapper component.
- Avoid nesting and pseudo-selectors in sx prop values when possible.

### The System CSS Props

For example, I would like my `Box` component to be able to accept not only the `sx` prop but also the space and typography CSS props. This could be done easily without much effort with `styled-system`.

<!-- prettier-ignore-start -->
```ts
import { 
  space, 
  SpaceProps, 
  typography, 
  TypographyProps,
} from "styled-system"

type StyledBoxProps = 
  SpaceProps & 
  TypographyProps & 
  SxProp

const Box = styled.div<StyledBoxProps>(
  space, 
  typography, 
  sx
)
```

Now, my `Box` component can accept spacing and typography props like this:

```tsx
<Box 
  mt={2} 
  p={1} 
  fontSize={3}
>
  the system CSS props works!
</Box>
```

<!-- prettier-ignore-end -->

## Enable Color Mode with CSS custom properties

`renderToString` renders a React tree to an HTML string.

The `styles()` are also rendered to string on the server side. And because there is no way to know the user's preference for color mode (except for using session), `styles()` will be rendered with the default color mode such as the light mode. This can cause two serious problems:

1. the `className`s mismatch during hydration
1. the flash of wrong styled content (well, only if you think it's serious)

Those problems could be solved by leveraging the CSS custom properties and one inline script.

1. create the global styles like this

   ```ts
   const lightColors = {
     "--wtlin-ui-colors-text": "white",
     "--wtlin-ui-colors-bg": "black",
   }
   const darkColors = {
     "--wtlin-ui-colors-text": "black",
     "--wtlin-ui-colors-bg": "white",
   }
   const GlobalStyle = css({
     html: {
       ...lightColors,
     },
     "html.wtlin-ui-light": lightColors,
     "html.wtlin-ui-dark": darkColors,
   })
   ```

2. style your component with CSS custom properties

   ```ts
   const Body = styled.div({
     color: "var(--wtlin-ui-colors-text)",
     background: "var(--wtlin-ui-colors-bg)",
   })
   ```

3. determine the color mode with inline script

   ```tsx
   <Script id="set-color-mode" strategy="beforeInteractive">
     {`
        try {
          let mode = localStorage.getItem('wtlin-ui-color-mode');
          if (!mode) {
            const preferDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
            mode = preferDarkMode ? 'dark' : 'light';
            localStorage.setItem('wtlin-ui-color-mode', mode)
          }
          if (mode) {
            document.documentElement.classList.add('wtlin-ui-' + mode);
          }
        } catch (e) {}
      `}
   </Script>
   ```

### CSS-in-JS without CSS custom properties

```

               server            client
                  │                 │
                  │     /home       │
              ┌───┤◄────────────────┤
         SSR  │   │                 │
 (light mode) │   │                 │
              └──►├────────────────►│
                  │                 ├───┐
                  │    /bundle.js   │   │ Construct DOM
                  │◄────────────────┤   │
                  │                 │◄──┘
                  ├────────────────►│
                  │                 ├───┐
                  │                 │   │ Construct CSSOM
                  │                 │   │
                  │                 │◄──┘
                  │                 │
                  │                 ├─ ─ ─ render page with ─ ─
                  │                 │        light mode      │
                  │                 ├───┐                    │
                  │                 │   │ Execute JS         │
                  │                 │   │ hydrate with     Flash!!
                  │                 │   │ dark mode          │
                  │                 │   │ (mismatch ERROR ❌)│
                  │                 │◄──┘                    │
                  │                 │                        ▼
                  │                 ├─ ─ ─ re-render with ─ ─ ─
                  │                 │        dark mode
                  │                 │
                  │                 │
                  │                 │
                 ─┴─               ─┴─

```

### CSS-in-JS with CSS custom properties

```

               server            client
                  │                 │
                  │     /home       │
              ┌───┤◄────────────────┤
         SSR  │   │                 │
              │   │                 │
              └──►├────────────────►│
                  │                 ├───┐
                  │    /bundle.js   │   │ Construct DOM
                  │◄────────────────┤   │ Execute inline script
                  │                 │◄──┘
                  ├────────────────►│
                  │                 ├───┐
                  │                 │   │ Construct CSSOM
                  │                 │   │
                  │                 │◄──┘
                  │                 │
                  │                 ├─ ─ ─ render page with ─ ─
                  │                 │         dark mode      │
                  │                 ├───┐                    │
                  │                 │   │ Execute JS         │
                  │                 │   │ hydrate with   No Flash!!
                  │                 │   │ dark mode          │
                  │                 │   │ (no ERROR ✅)      │
                  │                 │◄──┘                    │
                  │                 │                        ▼
                  │                 ├─ ─ ─ re-render with ─ ─ ─
                  │                 │        dark mode
                  │                 │
                  │                 │
                  │                 │
                 ─┴─               ─┴─

```

### Styled System with CSS custom properties

To make the theme clean as before without being messed up due to the CSS custom properties. I made two helper functions.

- `flatWithPath` - to put styles into html using CSS custom properties
- `toCssCustomProperties` - to use styled-system with CSS custom properties

So I can still define my theme like this:

```ts
import { flatWithPath, toCssCustomProperties } from "../theme"

const primitives = {
  garden: {
    flower: "red",
    stone: ["blue", "green", "yellow"],
    animal: {
      cat: "black",
      dog: "pink",
      rabbit: ["white", "cream"],
    },
  },
  bag: ["blue", "red", "yellow"],
  hair: "brown",
}

it("flats colors", () => {
  expect(flatWithPath(primitives, "--wtlin-ui-colors")).toStrictEqual({
    "--wtlin-ui-colors-garden-flower": "red",
    "--wtlin-ui-colors-garden-stone-0": "blue",
    "--wtlin-ui-colors-garden-stone-1": "green",
    "--wtlin-ui-colors-garden-stone-2": "yellow",
    "--wtlin-ui-colors-garden-animal-cat": "black",
    "--wtlin-ui-colors-garden-animal-dog": "pink",
    "--wtlin-ui-colors-garden-animal-rabbit-0": "white",
    "--wtlin-ui-colors-garden-animal-rabbit-1": "cream",
    "--wtlin-ui-colors-bag-0": "blue",
    "--wtlin-ui-colors-bag-1": "red",
    "--wtlin-ui-colors-bag-2": "yellow",
    "--wtlin-ui-colors-hair": "brown",
  })
})

it("maps colors to CSS custom properties", () => {
  expect(toCssCustomProperties(primitives, "--wtlin-ui-colors")).toStrictEqual({
    garden: {
      flower: "var(--wtlin-ui-colors-garden-flower)",
      stone: [
        "var(--wtlin-ui-colors-garden-stone-0)",
        "var(--wtlin-ui-colors-garden-stone-1)",
        "var(--wtlin-ui-colors-garden-stone-2)",
      ],
      animal: {
        cat: "var(--wtlin-ui-colors-garden-animal-cat)",
        dog: "var(--wtlin-ui-colors-garden-animal-dog)",
        rabbit: [
          "var(--wtlin-ui-colors-garden-animal-rabbit-0)",
          "var(--wtlin-ui-colors-garden-animal-rabbit-1)",
        ],
      },
    },
    bag: [
      "var(--wtlin-ui-colors-bag-0)",
      "var(--wtlin-ui-colors-bag-1)",
      "var(--wtlin-ui-colors-bag-2)",
    ],
    hair: "var(--wtlin-ui-colors-hair)",
  })
})
```

<!-- prettier-ignore-start -->
[emotion]: https://emotion.sh/docs/introduction
[styled-system]: https://styled-system.com/
[system-css-properties]: https://mui.com/system/properties/
[the-sx-prop]: https://mui.com/system/getting-started/the-sx-prop/
[prefers-color-scheme]: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme
[css-custom-properties]: https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties

<!-- emotion -->
[storybook-styled-component]: https://storybook.js.org/blog/541-components-from-styled-components-to-emotion/#:~:text=The%20team%20felt%20that%20Emotion%20offered%20better%20performance%20and%20features%20for%20our%20use%20case%20so%20we%20decided%20to%20refactor%20everything%20else%20to%20use%20Emotion.
[mui-styled-component]: https://mui.com/material-ui/guides/styled-engine/#:~:text=We%20strongly%20recommend%20using%20Emotion%20for%20SSR%20projects.
[atomic-css]: https://css-tricks.com/lets-define-exactly-atomic-css/
[airbnb-trip-to-linaria]: https://medium.com/airbnb-engineering/airbnbs-trip-to-linaria-dc169230bd12

<!-- styled system -->
[mui-the-sx-prop]: https://mui.com/system/getting-started/the-sx-prop/
[github-the-sx-prop]: https://primer.style/react/overriding-styles
[theme-ui-the-sx-prop]: https://theme-ui.com/sx-prop

<!-- prettier-ignore-end -->
