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
    neutral: "#545454",
    primary: "#1677FF",
    secondary: "#f759ab",
    error: "#ff4d4f",
    success: "#4ade80",
    warning: "#fb923c",
    info: "#22d3ee",
    highlight: "#facc15",
  },
  backgroundColor: "#ffffff",
  toneScale: {
    bold: 1.3,
    base: 1,
    soft: 0.5,
    subtle: 0.2,
    bare: 0.1,
    plain: 0
  },
}
```

Instead of hardcoding every color for each visual state, the Scaled Design System defines a base `backgroundColor` and a `palette`. Final UI colors are calculated using:

```ts
color = backgroundColor + (themeColor - backgroundColor) * toneScale
```

Each tone scale key — such as `plain`, `bare`, `soft`, `subtle`, `base`, `bold` — is a linear multiplier interpolating between `backgroundColor` and a palette color.

### Theme Output Colors

Auto-generated color tokens by semantic key:

| Semantic  | Bold    | Base        | Soft    | Subtle  | Bare    | Plain   |
| --------- | ------- | ----------- | ------- | ------- | ------- | ------- |
| Neutral   | #212121 | **#545454** | #aaaaaa | #dddddd | #eeeeee | #ffffff |
| Primary   | #004eff | **#1677ff** | #8bbbff | #d0e4ff | #e8f1ff | #ffffff |
| Secondary | #f52792 | **#f759ab** | #fbacd5 | #fddeee | #feeef7 | #ffffff |
| Error     | #ff181a | **#ff4d4f** | #ffa6a7 | #ffdbdc | #ffeded | #ffffff |
| Success   | #14d45a | **#4ade80** | #a5efc0 | #dbf8e6 | #edfcf2 | #ffffff |
| Warning   | #fa7102 | **#fb923c** | #fdc99e | #fee9d8 | #fff4ec | #ffffff |
| Info      | #00c6e9 | **#22d3ee** | #91e9f7 | #d3f6fc | #e9fbfd | #ffffff |
| Highlight | #f9bd00 | **#facc15** | #fde68a | #fef5d0 | #fffae8 | #ffffff |

## Color-Based Accessibility

By adjusting tones for `backgroundColor`, `color`, and `border`, we can automatically generate consistent and accessible states for any component.

### Tone Overview

* Hover: Slight change in background tone
* Focus: Border tone and border thickness change
* Active/Checked/Selected: Strong background tone change
* Disabled/Read-only: Lower contrast through subtle background and text color changes

### Color Mapping

* Focus: Use `primary` instead of the original color
* Selected/Checked: Use `primary` as the background
* Disabled/Read-only: Use `neutral` instead of the original color

## Visual Tone Table

### Normal

| Background | Color | Border |
| ---------- | ----- | ------ |
| plain      | base  | subtle |
| bare       | base  | soft   |
| subtle     | bold  | soft   |
| soft       | plain | base   |
| base       | plain | subtle |
| bold       | plain | subtle |

### Soft Background

| Background | Color | Border |
| ---------- | ----- | ------ |
| bare       | base  | subtle |
| subtle     | base  | soft   |
| soft       | bold  | soft   |
| base       | plain | base   |
| bold       | plain | subtle |

### Low Contrast

| Background | Color | Border |
| ---------- | ----- | ------ |
| bare       | soft  | soft   |
| subtle     | soft  | soft   |
| soft       | base  | base   |
| base       | bare  | bare   |
| bold       | bare  | bare   |

### Strong Border

| Background | Color | Border |
| ---------- | ----- | ------ |
| plain      | base  | soft   |
| bare       | base  | base   |
| subtle     | bold  | base   |
| soft       | plain | bold   |
| base       | plain | soft   |
| bold       | plain | soft   |

### Strong Background

| Background | Color | Border |
| ---------- | ----- | ------ |
| subtle     | bold  | subtle |
| soft       | plain | soft   |
| base       | plain | soft   |
| bold       | plain | base   |

### Visual State Mapping

| State        | Visual Tone       | Background  | Color       | Border      |
| ------------ | ----------------- | ----------- | ----------- | ----------- |
| normal       | Normal            | Input Color | Input Color | Input Color |
| visited      | Normal            | Input Color | Input Color | Input Color |
| hover        | Soft Background   | Input Color | Input Color | Input Color |
| readonly     | Soft Background   | Input Color | Input Color | Input Color |
| disabled     | Low Contrast      | **neutral** | **neutral** | **neutral** |
| focus        | Strong Border     | Input Color | Input Color | **primary** |
| invalid      | Strong Border     | Input Color | Input Color | **error**   |
| active       | Strong Background | Input Color | Input Color | Input Color |
| selected     | Strong Background | **primary** | Input Color | Input Color |
| checked      | Strong Background | **primary** | Input Color | **primary** |

All states are derived using tone adjustments — no hardcoded values.

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

## Dark Mode Strategy

Dark mode uses the same tone logic with a different `backgroundColor` and modified base palette. All tones update automatically using the same formula.

```ts
light.backgroundColor = "#ffffff"
dark.backgroundColor = "#000000"
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
  --soft-error:    #ffa6a7;
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
| `chunk` | Horizontal group of inlines (e.g., Button \[icon, text], InputText \[prefix...]) |
| `block` | Any component larger than `chunk`, usage nested composing blocks                                                 |

>After block, the system imposes no further layout rules.
Page-level composition such as columns, sections, margins, or responsive grid behavior are intentionally left unconstrained, allowing complete design flexibility without system interference.

### Visual Box
```
            InpuText  Chunk            Button Chunk
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
