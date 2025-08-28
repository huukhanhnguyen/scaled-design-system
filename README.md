# Scaled Design System

A compact, scalable, and mathematically predictable design specification that enables consistent and accessible design tokens for any implementation — without requiring verbose specs or additional tooling.

## Motivation

Most modern design systems are:

* Too large to onboard quickly
* Too rigid or tool-dependent to extend flexibly
* Too verbose to audit or automate
* Require a lot of manual work

## Design Goals

* How to make colors and sizes consistent
* How to make the system simple, flexible, and automatic
* How to write code without constantly referring to documentation
* Can we define a full design token system using just math and markdown

## Solutions

* **Tone-based color interpolation** instead of using many preset colors
* **Automatic accessibility for states and interactions** based on visual tone mapping
* **Box-level sizing based on font-size**, simulating how the brain groups elements visually

## Tone-based Color Scaling

### Theme Input Colors

```js
{
  palette: {
    neutral: "#5e5e5e",
    primary: "#1677FF",
    secondary: "#f759ab",
    error: "#ff4d4f",
    attention: "#ff6a4d",
    success: "#4ade80",
    warning: "#fb923c",
    info: "#22d3ee",
    highlight: "#facc15",
  },
  backgroundColor: "#ffffff",
  contrastColor: "#000000",
  toneScale: {
    plain: 0,
    bare: 0.1,
    slight: 0.2,
    gentle: 0.3,
    subtle: 0.4,
    origin: 0.5,
    bold: 0.65,
    intense: 0.8,
    extreme: 1,
  }
}
```
### Tones
- `plain → bare → slight →gentle→ subtle → origin → bold→ intense→ extreme`
- `plain` nearly to backgroundColor
- `extreme` nearly to contrastColor
- index define: next = previous + 1
### Color Formula
Instead of hardcoding every color for each visual state, the Scaled Design System defines simple `palette`. Final UI colors are calculated using:

```ts
let originScale = toneScale.origin
// toneScale == origin
color = themeColor
// toneScale < origin
color = backgroundColor + (themeColor - backgroundColor) * toneScale/originScale

// toneScale > origin
color = contrastColor + (contrastColor - themeColor) * (toneScale-originScale)/(1-originScale)
```
### Theme Output Colors

Auto-generated color tokens by semantic key:
| Tone       | Neutral   | Primary   | Secondary | Error     | Attention | Success   | Warning   | Info      | Highlight |
|------------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|
| `plain`    | #ffffff   | #ffffff   | #ffffff   | #ffffff   | #ffffff   | #ffffff   | #ffffff   | #ffffff   | #ffffff   |
| `bare`     | #dfdfdf   | #d0e4ff   | #fddeee   | #ffdbdc   | #ffe1db   | #dbf8e6   | #fee9d8   | #d3f6fc   | #fef5d0   |
| `slight`   | #bfbfbf   | #a2c9ff   | #fcbddd   | #ffb8b9   | #ffc3b8   | #b7f2cc   | #fdd3b1   | #a7edf8   | #fdeba1   |
| `gentle`   | #9e9e9e   | #73adff   | #fa9bcd   | #ff9495   | #ffa694   | #92ebb3   | #fdbe8a   | #7ae5f5   | #fce073   |
| `subtle`   | #7e7e7e   | #4592ff   | #f97abc   | #ff7172   | #ff8871   | #6ee599   | #fca863   | #4edcf1   | #fbd644   |
| `origin`     | #5e5e5e   | #1677ff   | #f759ab   | #ff4d4f   | #ff6a4d   | #4ade80   | #fb923c   | #22d3ee   | #facc15   |
| `bold`     | #424242   | #0f53b3   | #ad3e78   | #b33637   | #b34a36   | #349b5a   | #b0662a   | #1894a7   | #af8f0f   |
| `intense`  | #262626   | #093066   | #632444   | #661f20   | #662a1f   | #1e5933   | #643a18   | #0e545f   | #645208   |
| `extreme`  | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   |

- Output colors in format theme[tone][color]


### Background Tone Variants

| variants  | plain | bare | slight | gentle | subtle | origin | bold | intense | extreme |
| ------- | ----- | ---- | ------ | ------ | ------ | ---- | ---- | ------- | ------- |
| soft    | +1    | +1   | +1     | +1     | +1     | -1   | -1   | -1      | -1      |
| base | 0     | 0    | 0      | 0      | 0      | 0    | 0    | 0       | 0       |
| strong  | +2    | +2   | +2     | +2     | +2     | -2   | -2   | -2      | -2      |

- Example: `soft`at `plain` = plain   + 1 =  `bare`
### Text Color Tone Variants
Text tone  = BackgroundTone ± level to ensure readibility

| variants  | plain | bare | slight | gentle | subtle | origin | bold | intense | extreme |
| ------- | ----- | ---- | ------ | ------ | ------ | ---- | ---- | ------- | ------- |
| soft    | +3    | +3   | +3     | +3     | +4     | -3   | -3   | -3      | -3      |
| base | +4    | +4   | +4     | +4     | -3     | -4   | -4   | -4      | -4      |
| strong  | +5    | +5   | +5     | +5     | -4     | -5   | -5   | -5      | -5      |

### Stroke Color Tone Variants
- Stroke tone mean that it can use for border,box shadow, outline
- Stroke tone  = BackgroundTone ± level smaler than contrast of text

| adjust  | plain | bare | gentle | subtle | soft | origin | strong | bold | extreme |
| ------- | ----- | ---- | ------ | ------ | ---- | ---- | ------ | ---- | ------- |
| soft    | +1    | +1   | +1     | +1     | -1   | -1   | -1     | -1   | -1      |
| base | +2    | +2   | +2     | +2     | -2   | -2   | -2     | -2   | -2      |
| strong  | +3    | +3   | +3     | +3     | -2   | -3   | -3     | -3   | -3      |

### Color Variants in code
```
const backgroundTones = {
  soft: [1, 1, -1, 1, -1, -1, -1, -1, -1],
  base: [0, 0, 0, 0, 0, 0, 0, 0, 0],
  strong: [2, 2, 2, 2, -2, -2, -2, -2, -2],
} as const;

const textTones = {
  soft: [4, 4, 4, 4, -3, -4, -4, -4, -4, -4],
  base: [5, 5, 5, 5, -3, -4, -5, -5, -5],
  strong: [6, 6, 6, -3, -4, -5, -6, -6, -6],
} as const;

const strokeTones = {
  soft: [1, 1, 1, 1, -1, -1, -1, -1, -1],
  base: [2, 2, 2, 2, -2, -2, -2, -2, -2],
  strong: [3, 3, 3, 3, -2, -3, -3, -3, -3],
} as const;
```
## Multi Tone Strategy
The tones system work like theme-in-theme mean that we can use multi tone each page. 
Example in light theme we can use `plain` tone in origin and where `popover` we can use `origin` or more stronger to make dark mode. All state colors automatic compute so do not need manual adjustment for dark 'popup'
## Dark Mode Strategy

Dark mode uses the same tone logic with a different `backgroundColor` and modified origin palette. All tones update automatically using the same formula.

```ts
light.backgroundColor = "#ffffff"
light.contrastColor = "#5e5e5e"
light.palete.neutral = "#808080" // slighty near dark for make text more clearly

dark.backgroundColor = "#000000"
dark.contrastColor = "#ffffff"
light.palete.neutral = "#808080" // slighty near dark for make text more clearly
```

> Switching themes only requires changing `palette` and `backgroundColor`.

## Color Token Export Format

Tokens can be exported as CSS variables:

```css
:root {
  --plain-primary: #ffffff;
  --bare-primary:  #e8f1ff;
  --soft-primary:  #8bbbff;
  --origin-primary:  #1677ff;
  --bold-primary:  #004eff;
  ...
}
```

With theme namespaces:

```css
:root[data-theme="light"] {
  --plain-primary: #ffffff;
}
:root[data-theme="dark"] {
  --plain-primary: #0d1117;
}
```

## Line Height Based Layout 
> Any component dimensions must relative to line height easy to make Vertical rhythm and composing components

### Sixth Fraction Line Height

Why 1/6 lineHeight is best unit

- Vertical rhythm => most important is paddingY => need minimum value 3-4px
- fontSize usually 14px - 16px => lineHeight 21px - 24px => 1/6 ~ 3.5px - 4px
- lineHeight usually ~ 1.5 em => fontSize ~ 2/3 lineHeight ~ 4/6 lineHeight

```js
lineSpan6: { // n*(lineHeight*em)/6
  span1: "0.25em",
  span2: "0.5em",
  span3: "0.75em",
  span4: "1em",
  span5: "1.25em",
  span6: "1.5em"
},
```

* All units use `em` to scale with `fontSize`
* Divide lineHeight to 6 spans
* `span*` use for `padding` `borderRadius` `margin` `gap`
* Larger use `calc(n * ${lineSpan6.span6})`
* PaddingX must larger or equals paddingY
* Border radius must equals paddingY
* Children gap/margin must between paddingY and paddingX

### Component Types

| Component Types   | Description                                                                    |
| ------- | ------------------------------------------------------------------------------ |
| `inline`  | text/checkbox/radio/icon/tag/chip/caption/badge, etc. — smallest unit , `padding`=`borderRadius`=0, `margin` `gap` = span2/span3                    |
| `bar` | Horizontal group of inlines (e.g., Button,InputText,Select), span1/span2/span3 use for `padding` `borderRadius` `margin` `gap`  |
| `block` | Any component larger than `bar`use span3 and `calc(n * ${lineSpan6.span6})` for `padding` `borderRadius` `margin` `gap`=> try make rounded total heigt = n*lineHeight                                           |

## Typography

### Typography Input

```js
fontSize: {
        h6: "1.3rem",
        h5: "1.4rem",
        h4: "1.5rem",
        h3: "1.6rem",
        h2: "1.7rem",
        h1: "1.8rem",
        sm: "0.8rem",
        md: "1rem",
        lg: "1.2rem",
    },
    lineHeight: {
        h6: 1.6,
        h5: 1.5,
        h4: 1.4,
        h3: 1.3,
        h2: 1.2,
        h1: 1.1,
        sm: 1.5,
        md: 1.5,
        lg: 1.5,
    },
    fontWeight: {
        h1: 700,
        h2: 700,
        h3: 700,
        h4: 600,
        h5: 600,
        h6: 500,
        sm: 400,
        md: 400,
        lg: 400,
    },
fontFamily: {
  heading: '"Roboto", "Arial", sans-serif',
  text: '"Roboto","Inter", "Helvetica Neue", sans-serif',
  mono: '"Courier New", monospace',
},
```

### Typography Scaling

Sizes like `sm`, `md`, `lg` define component variants. Combined with `em`-based layout sizes, components scale automatically.

### Typography Token Export

```css
:root {
  --fontSize-sm: 0.85rem;
  --fontSize-md: 1rem;
  --fontSize-lg: 1.1rem;
  ...
}
```

## Use Cases

* **Framework-agnostic**: Works with React, Vue, or plain HTML/CSS
* **Accessible by base**: Tone-based contrast mapping
* **Scalable layout**: Consistent spacing on all screen sizes
* **SSR-friendly**: Outputs CSS directly


## Comparison to Other Systems

### Google Material Design

| Feature               | Material Design     | Scaled Design System           |
| --------------------- | ------------------- | ------------------------------ |
| **Token Granularity** | High (1000+)        | Minimal & scalable             |
| **Color Definition**  | Predefined palettes | Auto-generated with tone logic |
| **Visual States**     | Manually defined    | Dynamically calculated         |
| **Spec Length**       | 100s of pages       | One markdown file              |
| **Accessibility**     | Supported           | Built-in tone contrast logic   |

> Scaled Design System is a minimalist alternative — not a clone. It achieves flexibility through math instead of verbosity.

### IBM Carbon Design System

| Feature             | Carbon System        | Scaled Design System |
| ------------------- | -------------------- | -------------------- |
| **Layout Grid**     | 2x/4x Grid           | Box-level scaling    |
| **Color Themes**    | Predefined + tooling | Palette × tone math  |
| **Component Scope** | Full UI kit          | Token logic only     |
| **CSS Variables**   | ✔️                   | ✔️                   |
| **Spec Size**       | 100+ pages + tooling | One markdown file    |

> Scaled Design System is not a UI kit replacement. It offers a predictable, composable token-generation strategy.

## Implementation

* [Domphy Theme](https://github.com/domphy/theme)

License: MIT Copyright 2025 Author: [Khánh Nguyễn](https://github.com/huukhanhnguyen)
