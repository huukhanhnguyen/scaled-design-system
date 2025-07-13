# Scaled Design System

A compact, scalable, and mathematically predictable design specification enables consistent and accessible design tokens for any implementation — without needing verbose specs or extra tooling.

## Motivation

Most modern design systems are:

* Too large to onboard quickly
* Too rigid or tool-dependent to extend flexibly
* Too verbose to audit or automate

This specification began with a question: **Can we define a full design token system using just math and markdown?**

## Specification Goals

* Describe **tone-based color interpolation**
* Describe **font-size-scaled layout units**
* Define **a minimal set of state tone mappings**
* Export **CSS variables**
* Require **no tools or build steps**
* Be expressed fully in one markdown document

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

Instead of hardcoding every visual state color, the Scaled Design System defines a base `backgroundColor` and a color `palette`. The final UI colors are calculated by:

```ts
color - backgroundColor + (themeColor - backgroundColor) * toneScale
```

Each tone scale key — such as `plain`, `bare`, `soft`, `subtle`, `base`, `bold` — is a linear multiplier that interpolates between `backgroundColor` and a palette color.

### Theme Output Colors
Auto geneation from `tone scale`
| Semantic  | Bold    | Base    | Soft    | Subtle  | Bare    | Plain   |
| --------- | ------- | ------- | ------- | ------- | ------- | ------- |
| Neutral   | #212121 | #545454 | #aaaaaa | #dddddd | #eeeeee | #ffffff |
| Primary   | #004eff | #1677ff | #8bbbff | #d0e4ff | #e8f1ff | #ffffff |
| Secondary | #f52792 | #f759ab | #fbacd5 | #fddeee | #feeef7 | #ffffff |
| Error     | #ff181a | #ff4d4f | #ffa6a7 | #ffdbdc | #ffeded | #ffffff |
| Success   | #14d45a | #4ade80 | #a5efc0 | #dbf8e6 | #edfcf2 | #ffffff |
| Warning   | #fa7102 | #fb923c | #fdc99e | #fee9d8 | #fff4ec | #ffffff |
| Info      | #00c6e9 | #22d3ee | #91e9f7 | #d3f6fc | #e9fbfd | #ffffff |
| Highlight | #f9bd00 | #facc15 | #fde68a | #fef5d0 | #fffae8 | #ffffff |

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

* ±1 tone Background
* Color and Border unchanged

| Background | Color | Border |
| ---------- | ----- | ------ |
| bare       | base  | subtle |
| subtle     | base  | soft   |
| soft       | bold  | soft   |
| base       | plain | base   |
| bold       | plain | subtle |

### Low Contrast

* ±1 tone Background
* ±1 tone Color
* tone Color equals Border

| Background | Color | Border |
| ---------- | ----- | ------ |
| bare       | soft  | soft   |
| subtle     | soft  | soft   |
| soft       | base  | base   |
| base       | bare  | bare   |
| bold       | bare  | bare   |

### Strong Border

* ±1 tone Border
* Color and Background unchanged

| Background | Color | Border |
| ---------- | ----- | ------ |
| plain      | base  | soft   |
| bare       | base  | base   |
| subtle     | bold  | base   |
| soft       | plain | bold   |
| base       | plain | soft   |
| bold       | plain | soft   |

### Strong Background

* ±1 tone Background
* Color and Border unchanged

| Background | Color | Border |
| ---------- | ----- | ------ |
| subtle     | bold  | subtle |
| soft       | plain | soft   |
| base       | plain | soft   |
| bold       | plain | base   |

## Visual State Mapping

| State        | Visual Tone       | Background  | Color       | Border      |
| ------------ | ----------------- | ----------- | ----------- | ----------- |
| normal       | Normal            | Input Color | Input Color | Input Color |
| visited      | Normal            | Input Color | Input Color | Input Color |
| hover        | Soft Background   | Input Color | Input Color | Input Color |
| readonly     | Soft Background   | Input Color | Input Color | Input Color |
| **disabled** | Low Contrast      | **neutral** | **neutral** | **neutral** |
| focus        | Strong Border     | Input Color | Input Color | **primary** |
| invalid      | Strong Border     | Input Color | Input Color | **error**   |
| active       | Strong Background | Input Color | Input Color | Input Color |
| selected     | Strong Background | **primary** | Input Color | Input Color |
| checked      | Strong Background | **primary** | Input Color | **primary** |

### Auto-compute Colors for Each State

With input color and tone we can use mapping to compute colors by steps:

* Each state can define tone of `background`, `color`, and `border`
* Based on Theme Output we know the values
* The result is computed and used as CSS variables

```ts
{
  default: { backgroundColor: "--plain-primary", ... },
  hover:   { backgroundColor: "--bare-primary", ... },
  focus:   { borderColor: "--soft-primary", ... },
  disabled: { backgroundColor: "--bare-neutral", ... },
  ...
}
```

All states are resolved by tone offsets — no hardcoding.

## Dark Mode Strategy

Dark mode uses the same tone logic with a different `backgroundColor` and adjusted palette base colors. All derived tones auto-update by the same formula.

```ts
light.backgroundColor = "#ffffff"
dark.backgroundColor = "#000000"
```

Tone logic ensures consistent visual contrast across themes.

> Switching themes does not require redefining all variables. Only base inputs (`palette`, `backgroundColor`) change.

## Token Export Format

Each computed token can be exported as CSS custom properties:

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

You can namespace by theme:

```css
:root[data-theme="light"] {
  --plain-primary: #ffffff;
}
:root[data-theme="dark"] {
  --plain-primary: #0d1117;
}
```

## Box-level Spacing & Sizing

Define your layout by logical `boxSize` units:

| Token   | Description                                                                          |
| ------- | ------------------------------------------------------------------------------------ |
| `cell`  | Input(checkbox, radio, text, select), text, icons, smallest unit                     |
| `chunk` | Horizontal group of cells, e.g., Button\[icon,text], InputText\[prefix,input,suffix] |
| `row`   | Horizontal group of chunks, e.g., Field + InputText                                  |
| `block` | Card or modal containing rows                                                        |
| `panel` | Contains blocks                                                                      |
| `page`  | Contains panels/blocks                                                               |

Each unit applies to `spacing`, `padding`, and `borderRadius` using scalable `em` values tied to `fontSize`.

> ✅ `panel`, `block`, `row` can be **nested recursively**
> ✅ Manual padding exceptions (e.g., main content) allowed

## Use Cases

* **Framework-agnostic theming**: Works in React, Vue, plain HTML/CSS
* **Accessible design by default**: Contrast-aware tone mapping
* **Scalable spacing**: Layout consistency across devices & components
* **Server-side rendering**: Generates CSS directly

---

## Comparison

### Google Material Design

| Feature               | Material Design     | Scaled Design System            |
| --------------------- | ------------------- | ------------------------------- |
| **Token Granularity** | High (over 1,000)   | Minimal & scalable              |
| **Color Definition**  | Predefined palettes | Auto-generated by tone math     |
| **Visual States**     | Manually listed     | Dynamically calculated          |
| **Spec Length**       | 100s of pages       | One .md file                    |
| **Accessibility**     | Supported           | Built-in by tone contrast logic |

> Scaled Design System aims to be a minimalist alternative — not a clone. It achieves similar flexibility using scalable math, not verbosity.

### Carbon Design System (IBM)

| Feature             | Carbon System        | Scaled Design System |
| ------------------- | -------------------- | -------------------- |
| **Layout Grid**     | 2x/4x Grid           | Box-level scaling    |
| **Color Themes**    | Predefined + tools   | Palette × tone math  |
| **Component Scope** | Full UI kit          | Token logic only     |
| **CSS Variables**   | ✔️                   | ✔️                   |
| **Spec Size**       | 100+ pages + tooling | One markdown spec    |

> Scaled Design System doesn't aim to replace Carbon's UI components, only to offer a composable token-generation strategy.

## Implementation

* [Domphy Theme](https://github.com/domphy/theme)
  
License: MIT Copyright: 2025 Author: [Khánh Nguyễn](https://github.com/huukhanhnguyen)
