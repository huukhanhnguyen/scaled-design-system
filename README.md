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
    base: 0.5,
    bold: 0.65,
    intense: 0.8,
    extreme: 1,
  }
}
```
### Tones
- `plain → bare → slight →gentle→ subtle → base → bold→ intense→ extreme`
- `plain` nearly to backgroundColor
- `extreme` nearly to contrastColor
- index define: next = previous + 1
### Color Formula
Instead of hardcoding every color for each visual state, the Scaled Design System defines simple `palette`. Final UI colors are calculated using:

```ts
let baseScale = toneScale.base
// toneScale == base
color = themeColor
// toneScale < base
color = backgroundColor + (themeColor - backgroundColor) * toneScale/baseScale

// toneScale > base
color = contrastColor + (contrastColor - themeColor) * (toneScale-baseScale)/(1-baseScale)
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
| `base`     | #5e5e5e   | #1677ff   | #f759ab   | #ff4d4f   | #ff6a4d   | #4ade80   | #fb923c   | #22d3ee   | #facc15   |
| `bold`     | #424242   | #0f53b3   | #ad3e78   | #b33637   | #b34a36   | #349b5a   | #b0662a   | #1894a7   | #af8f0f   |
| `intense`  | #262626   | #093066   | #632444   | #661f20   | #662a1f   | #1e5933   | #643a18   | #0e545f   | #645208   |
| `extreme`  | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   | #000000   |

- Output colors in format theme[tone][color]


### Background Tone Variants

| variants  | plain | bare | slight | gentle | subtle | base | bold | intense | extreme |
| ------- | ----- | ---- | ------ | ------ | ------ | ---- | ---- | ------- | ------- |
| soft    | +1    | +1   | +1     | +1     | +1     | -1   | -1   | -1      | -1      |
| default | 0     | 0    | 0      | 0      | 0      | 0    | 0    | 0       | 0       |
| strong  | +2    | +2   | +2     | +2     | +2     | -2   | -2   | -2      | -2      |

- Example: `soft`at `plain` = plain   + 1 =  `bare`
### Text Color Tone Variants
Text tone  = BackgroundTone ± level to ensure readibility

| variants  | plain | bare | slight | gentle | subtle | base | bold | intense | extreme |
| ------- | ----- | ---- | ------ | ------ | ------ | ---- | ---- | ------- | ------- |
| soft    | +4    | +4   | +4     | +4     | +4     | -3   | -4   | -4      | -4      |
| default | +5    | +5   | +5     | +5     | -3     | -4   | -5   | -5      | -5      |
| strong  | +6    | +6   | +6     | -3     | -4     | -5   | -6   | -6      | -6      |

### Border Color Tone Variants
- Border tone  = BackgroundTone ± level smaler than contrast of text

| adjust  | plain | bare | gentle | subtle | soft | base | strong | bold | extreme |
| ------- | ----- | ---- | ------ | ------ | ---- | ---- | ------ | ---- | ------- |
| soft    | +1    | +1   | +1     | +1     | -1   | -1   | -1     | -1   | -1      |
| default | +2    | +2   | +2     | +2     | -2   | -2   | -2     | -2   | -2      |
| strong  | +3    | +3   | +3     | +3     | -2   | -3   | -3     | -3   | -3      |

### Color Variants in code
```
const backgroundTones = {
  soft: [1, 1, -1, 1, -1, -1, -1, -1, -1],
  default: [0, 0, 0, 0, 0, 0, 0, 0, 0],
  strong: [2, 2, 2, 2, -2, -2, -2, -2, -2],
} as const;

const textTones = {
  soft: [4, 4, 4, 4, -3, -4, -4, -4, -4, -4],
  default: [5, 5, 5, 5, -3, -4, -5, -5, -5],
  strong: [6, 6, 6, -3, -4, -5, -6, -6, -6],
} as const;

const borderTones = {
  soft: [1, 1, 1, 1, -1, -1, -1, -1, -1],
  default: [2, 2, 2, 2, -2, -2, -2, -2, -2],
  strong: [3, 3, 3, 3, -2, -3, -3, -3, -3],
} as const;
```
### Component State Colors

| states | background | text    | border  | color     | description        |
| ---------------- | ---------- | ------- | ------- | --------- | ------------------ |
| default          | default    | default | default | theme     | Normal appearance  |
| hover            | soft       | soft    | default | theme     | Hover effect       |
| active           | strong     | strong  | default | theme     | Pressed state      |
| selected         | strong     | strong  | default | primary   | Selected item      |
| checked          | strong     | default | strong  | primary   | Checked state      |
| readonly         | default    | soft    | soft    | neutral   | Read-only element  |
| focus            | default    | default | strong  | primary   | Focused element    |
| invalid          | default    | default | strong  | error     | Invalid input      |
| disable          | soft       | soft    | soft    | neutral   | Disabled state     |
| visited          | default    | default | default | secondary | Visited link       |
| current          | default    | default | strong  | primary   | Active/current tab |

- `current` is custom state then use with `aria-current` attributes and css, it need for distingush with `selected` form behaviour

### Auto-computed State Colors
So with  `themeColor` and `themeTone` inputs we can compute `backgroundColor` `color` `borderColor` of each states any components
Example:

```ts
{
  default: { backgroundColor: "--plain-primary", ... },
  hover:   { backgroundColor: "--bare-primary", ... },
  focus:   { borderColor: "--soft-primary", ... },
  disabled: { backgroundColor: "--bare-neutral", ... },
  ...
}
```
> low level usage can use with direct variant of colors, each properties {color,borderColor,backgroundColor} has {default,soft,strong}  variant. Example input value must `strong`, `placeholder` must `soft` and `label` must `default`. It make slightly distingush with thout depend on font weight and bold (too strong)
## Multi Tone Strategy
The tones system work like theme-in-theme mean that we can use multi tone each page. 
Example in light theme we can use `plain` tone in main and where `popover` we can use `base` or more stronger to make dark mode. All state colors automatic compute so do not need manual adjustment for dark 'popup'
## Dark Mode Strategy

Dark mode uses the same tone logic with a different `backgroundColor` and modified base palette. All tones update automatically using the same formula.

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
  --base-primary:  #1677ff;
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

## Box-level Layout

### Theme Layout Input

```js
borderRadius: {
    inline: "0.2em",
    chunk: "0.4em",
    block: "0.6em",
  },
  gap: {
    inlineBox: "0.3em",// readonly
    inline: "0.4em",
    chunk: "0.6em",
    block: "1.2em",
  },
  padding: {
    inline: "0em",// readonly must be 0
    chunk: "0.25em 0.75em",
    block: "0.6em 0.6em",
  }
```

* All units use `em` to scale with `fontSize`
* `padding`: padding of current box
* `gap`: spacing between children
* `borderRadius`: border-radius

### Box Size Definitions

| Token   | Description                                                                    |
| ------- | ------------------------------------------------------------------------------ |
| `inline`  | text/checkbox/radio/icon/tag/chip/caption/badge, etc. — smallest unit                    |
| `bar` | Horizontal group of inlines (e.g., Button,InputText,Select,) |
| `block` | Any component larger than `chunk`, usage nested composing blocks                                                 |

>After block, the system imposes no further layout rules.
Page-level composition such as columns, sections, margins, or responsive grid behavior are intentionally left unconstrained, allowing complete design flexibility without system interference.

### Visual Box
```
            InpuText Bar            Button Bar
        +------------------------+  +-------------------+
Block   | prefix | input | suffix|  |  icon   |   text  |
        +------------------------+  +-------------------+
```
```
+----------------------------------------------+
|                  Block                       |
|  +----------------------------------------+  |
|  |                Block                   |  |
|  +----------------------------------------+  |
|  +----------------------------------------+  |
|  |                Block                   |  |
|  +----------------------------------------+  |
|  +----------------------------------------+  |
|  |                Block                   |  |
|  +----------------------------------------+  |
+----------------------------------------------+
```

## Typography

### Typography Input

```js
fontSize: {
  h6: "1.1rem",
  h5: "1.2rem",
  h4: "1.3rem",
  h3: "1.5rem",
  h2: "1.75rem",
  h1: "2rem",
  sm: "0.85rem",
  md: "1rem",
  lg: "1.1rem",
},
lineHeight: {
  h6: 1.2,
  h5: 1.3,
  h4: 1.5,
  h3: 1.75,
  h2: 2,
  h1: 2.5,
  sm: 1.1,
  md: 1.4,
  lg: 1.6,
},
fontWeight: {
  h6: 500,
  h5: 500,
  h4: 500,
  h3: 700,
  h2: 700,
  h1: 700,
  sm: 500,
  md: 500,
  lg: 500,
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
* **Accessible by default**: Tone-based contrast mapping
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
