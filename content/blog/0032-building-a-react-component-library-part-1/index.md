---
title: Building a React Component Library - Part 1
excerpt: building-a-react-component-library-part1
date: 2023-02-22
tags: [2023, ui-library, color-mode]
slug: building-a-react-component-library-part1
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

When it comes to React component libraries, there's no shortage of excellent options to choose from. Popular libraries like `material-ui`, `react-spectrum`, `fluentui`, and `polaris` have already earned their place as favorites among developers. Nevertheless, as I plan for an upcoming website project, I can't shake the idea of creating my own library from scratch. In this blog post, I'll explain how and why I made certain technical decisions for my new library, `wtlin-ui`.

These decisions include choosing [Emotion][emotion] as the style engine, implementing theme-based [system CSS props][system-css-properties] and [`sx` Prop][the-sx-prop] with the [Styled System][styled-system] library, and providing theme colors via [CSS custom properties][css-custom-properties] to enable [Color Mode][prefers-color-scheme] without the dreaded flash of unstyled content.

## Choose `Emotion` as the style engine

When it comes to styling frontend applications, there are several popular options available, including CSS, SASS, CSS-in-JS, and zero-runtime libraries. As I considered my choices, I narrowed my focus to two CSS-in-JS libraries (styled-components and Emotion) and two zero-runtime libraries (`Linaria` and `vanilla-extract`).

Out of these four options, `styled-components` is undoubtedly one of the most well-established and widely used libraries, with extensive documentation. However, I ultimately decided to use `Emotion` instead. `Emotion` has a smaller bundle size and offers superior performance and more flexibility, making it the ideal choice for my project. In fact, the Storybook and MUI teams both recommend using `Emotion` for various use cases.

`Linaria` was another library that caught my attention. This zero-runtime CSS-in-JS library extracts CSS to standalone CSS files during build time, eliminating the need for runtime CPU overhead. This approach also offers caching benefits and makes it possible to deduplicate styles using [Atomic CSS][atomic-css] CSS. However, the lack of top-notch documentation and dedicated website, combined with the trial-and-error process of finding information, ultimately led me away from choosing this library.

Finally, I considered `vanilla-extract`, a modern solution with excellent TypeScript integration and no runtime overhead. While its minimal features, straightforwardness, and opinionated nature appealed to me, the fact that it processes everything at compile time and generates static CSS files was not enough to overcome the significant downside of code co-location.

Ultimately, after careful consideration, I decided to go with `Emotion` for its superior performance, flexibility, and overall suitability for my project's needs.

## Add theme-based system CSS props and `sx` prop with Styled System

System CSS properties and the sx prop are becoming increasingly popular. Many frameworks, like `material-ui`, support them nowadays. I personally use them as they simplify components and prevent developers from jumping back and forth to check trivial layout styles.

Writing code like this whenever you want to retrieve values from the theme can become tedious:

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

### The theme-based Style Props

Style functions of `styled-system` will try to find a value from the theme object, even for deeply nested values, and fallback to a hard-coded value if they can't.

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

// renders CSS `50%` width since it's not defined in the theme
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

Next, combine it with `styled` API provided by `emotion` (or `styled-component`):

```ts
import styled from "@emotion/styled"
import sx, { SxProp } from "./sx"

type StyledBoxProps = SxProp
const Box = styled.div<StyledBoxProps>(sx)
```

Now, the `Box` component can accept `sx` prop and be used like this:

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

While the `sx` prop is powerful, it's important not to abuse it. Here are some guidelines:

- Use the `sx` prop for small stylistic changes to components. For more substantial changes, consider abstracting your style changes into your own wrapper component.
- Avoid nesting and pseudo-selectors in `sx` prop values when possible.

### The System CSS Props

Suppose I want my `Box` component to accept not only the `sx` prop but also the `space` and `typography` CSS props. In that case, it can be done easily with `styled-system`:

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

The `renderToString` method renders a React tree to HTML string, including the `styles()` that are also rendered to string on the server-side. However, as there is no way to know the user's preference for color mode (apart from using a session), styles() are rendered with the default color mode, which is typically the light mode. This can result in two significant problems:

1. the mismatching of content during hydration
1. the flash of incorrectly styled content (which may not be critical but can still be problematic)

Fortunately, these issues can be resolved by taking advantage of CSS custom properties and an inline script.

1. Create global styles like this:

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

2. Style your component with CSS custom properties:

   ```ts
   const Body = styled.div({
     color: "var(--wtlin-ui-colors-text)",
     background: "var(--wtlin-ui-colors-bg)",
   })
   ```

3. Determine the color mode with inline script

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

To make the theme clean and avoid being messed up by CSS custom properties, I created two helper functions: `flatWithPath` and `toCssCustomProperties`.

With these helper functions, I can define the theme in the same way as before:

```ts
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
```

The `flatWithPath` function is used to apply the styles to the HTML using CSS custom properties, while the `toCssCustomProperties` function enables the use of `styled-system` with CSS custom properties.

Here's an example of how these functions can be used:

```ts
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

With these functions, the theme is kept clean and separate from the use of CSS custom properties, making it easier to maintain and modify the theme as needed.

## Reference

- [Storybook Team - How we migrated 541 components from Styled Components to Emotion][storybook-styled-component]
- [MUI Team - We strongly recommend using Emotion for SSR projects][mui-styled-component]
- [Airbnb Team - Airbnb’s Trip to Linaria][airbnb-trip-to-linaria]
- [MUI Team - The sx prop][mui-the-sx-prop]
- [GitHub Team - The sx prop][github-the-sx-prop]
- [Theme UI - The sx prop][theme-ui-the-sx-prop]

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
