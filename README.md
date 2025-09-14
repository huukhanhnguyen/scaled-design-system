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
* **Sixth line height fraction for spacing (margin,padding,radius,gap)

## Tone-based Color Scaling

### Theme Input Colors

LightTheme

```js
{
  palette: {
        neutral: "#7f7a7f",
        surface: "#a8a8a8",
        primary: "#1677FF",
        secondary: "#f759ab",
        error: "#ff4d4f",
        attention: "#ff6a4d",
        success: "#1bab50",
        warning: "#fb923c",
        info: "#22d3ee",
        highlight: "#ffcc00",
    },
    backgroundColor: "#ffffff",
    contrastColor: "#000000",
    toneScale: {
        plain: 0, // white
        bare: 0.05,
        slight: 0.125,
        subtle: 0.225,
        soft: 0.35,
        base: 0.5,
        sharp: 0.55,
        strong: 0.625,
        bold: 0.725,
        intense: 0.85,
        extreme: 1, // black
    },
}
```
Darktheme inherit LightTheme and update

```js
{
    backgroundColor: "#000000",
    contrastColor: "#ffffff",
    toneScale: {
        plain: 0, // black
        bare: 0.15,
        slight: 0.275,
        subtle: 0.375,
        soft: 0.45,
        base: 0.5,
        sharp: 0.65,
        strong: 0.775,
        bold: 0.875,
        intense: 0.95,
        extreme: 1, // white
    }
}

```

### Tones
- `plain → bare → slight → subtle → soft → base → sharp → strong → bold→ intense→ extreme`
- `plain` nearly to backgroundColor
- `extreme` nearly to contrastColor

> Why 11 color scales ? Excludes (plain/extreme) => 9 real color scales. Reference `Chakra` and some ui lib has arround 11 color scale but them has not perceptual visual difference for human eyes => 9 real color scales best fit

> Delta of scale increase when color darker for clearer perceptual contrast difference

### Color Formula
Instead of hardcoding every color for each visual state, the Scaled Design System defines simple `palette`. Final UI colors are calculated using:

> number of colors = palette * toneScale

```ts
let scale = toneScale.base
color = themeColor
// toneScale < scale
color = backgroundColor + (themeColor - backgroundColor) * toneScale/scale

// toneScale > scale
color = contrastColor + (contrastColor - themeColor) * (toneScale-scale)/(1-scale)
```

### Balance Tone Function

- At middle tone scale `base` (input color) => has 2 side of scales
- At start side shift forward and end side shift backward
- Instead turn direction at middle (index 6) => turn early at (index 5) => expected human likely on filled area text must same color outer area. Example button with color primary(or any) at tone basefilled as input color => text ±6 backforward to `plain` (same as background) instead forward `extreme` (black)

- Ending ensure clamp tone 

1. Instead move forware and invert at middle (index 6) => early invert at  
- Auto clamp when out of range
```js

export function balanceTone(tone, level) {
    const ThemeTones = ["plain","bare","slight","subtle","soft","base","sharp","strong","bold","intense","extreme"] 
    const index = ThemeTones.findIndex((e) => e === tone);
    if (index === -1) return tone;
    // invert point at index 5 ensure text nearly background like human expected
    let newIndex = index <= 5 ? index + level : index - level
    newIndex = newIndex < 0 || newIndex > ThemeTones.length - 1 ? - newIndex : newIndex
    newIndex = Math.max(0, Math.min(ThemeTones.length - 1, newIndex));
    return ThemeTones[newIndex];
}
```

### Self Tone Shift
The concept is that any element has `theme:{color,tone}` and default them are `inherit`. `theme.tone` mean that it self `background tone` when default state. Any `color` still work fine and we only care about `tone`.

- text tone= background ±6 
- subtext= background ±5
- emphasizedText= background ±7
- border/outline/boxShadow = background ±3
- hoverBackground = background ±1
- disabledBackground = background ±1
- disabledText = disabledBackground ±3 (reduce contrast)
- selectedBackground = background ±1 and change color 
- or selectedBackground = background ±1 or any change color 

> No need use `fontWeigth` (too contrast) for multiple semantic text, color tone more subtle

>By changing color and tone of `backgroundColor` `text` `border/outline/boxShadow` => apply any states

### Inherit Tone Shift

This for solving highlight children. Their tone must shift base on parent, it self tone auto change by above principles

>highlightTone = parentTone ±n (n >1 ensure parent hover or selected still see difference)

Examples: 

- tooltip/popover/badge tone = parentTone ±6
- code/tag/chip/switch tone = parentTone ±2

### Special Tone Case

Palette `neutral` with 11 tones use for almost of elements. But in some case need slightly tone and it might out of tone range. Example `pre` has larg area neend too low contrast  => Use other gray color, above palette has `surfure` it lower than `neutral`

> Out of tone range make other color 

## Color Token Export Format
Value of element should use css variables like element.style.color = var(--bare-primary) Container set attributes "data-theme="light" or data-theme="dark" => auto change color of element where use css variable
```css
:[data-theme="light"] {
  --bare-primary: #d1e4ff; //near white
}
:[data-theme="dark"] {
  --soft-primary: #02204d; // near dark
}
```
> Edge case why use same color in multiple theme => use direct token like "#d1e4ff"

## Line Height Based Layout 
> Any component dimensions must relative to line height easy to make Vertical rhythm and composing components

### Sixth Fraction Line Height

Why 1/6 lineHeight is best unit

- Vertical rhythm => most important is paddingY => need small value 3-4px
- fontSize usually 14px - 16px => lineHeight 21px - 24px => 1/6 ~ 3.5px - 4px
- lineHeight usually ~ 1.5 em => fontSize ~ 2/3 lineHeight ~ 4/6 lineHeight
- minimum "1/12": "0.125em" ~2px => border/offset 
```js
    lineStep: { //n*lineHeight(1.5em)
        "1/12": "0.125em",
        "1/6": "0.25em",
        "2/6": "0.5em",
        "3/6": "0.75em",
        "4/6": "1em",
        "5/6": "1.25em",
        "6/6": "1.5em",
        "9/6": "2.25em",
        "12/6": "3em",
        "18/6": "4.5em",
        "24/6": "6em",
        "30/6": "7.5em",
        "36/6": "9em",
        "42/6": "10.5em",
        "48/6": "12em",
        "54/6": "13.5em",
        "60/6": "15em",
    },
```
CSS Variables
```css
--lineStep-1_12: 0.125em;
--lineStep-1_6: 0.25em;
--lineStep-2_6: 0.5em;
```
* All units use `em` to scale with `fontSize`
* Divide lineHeight to 6 spans
* `span*` use for `padding` `borderRadius` `margin` `gap`
* Larger use `calc(n * ${lineHeight6.span6})`
* PaddingX must larger or equals paddingY
* Border radius must equals paddingY
* Children gap/margin must between paddingY and paddingX

### Component Types

| Component Types   | Description                                                                    |
| ------- | ------------------------------------------------------------------------------ |
| `inline`  | text/checkbox/radio/icon/tag/chip/caption/badge, etc. — smallest unit , `padding`=`borderRadius`=0, `margin` `gap` = span2/span3                    |
| `bar` | Horizontal group of inlines (e.g., Button,InputText,Select), span1/span2/span3 use for `padding` `borderRadius` `margin` `gap`  |
| `block` | Any component larger than `bar`use span3 and `calc(n * ${lineHeight6.span6})` for `padding` `borderRadius` `margin` `gap`=> try make rounded total heigt = n*lineHeight                                           |

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
## Usage Workflow 

* Each element define `theme={color,tone,size}`
* `{color,tone,size}` each can be specific value or "inherit"|"increase-{n}"|"descrease-{n}"|"balance-{n}"
* Traverse up tree to resolve inherit value `{color,tone,size}`
* Still dynamic tone by `balanceTone` + color => result color apply for each css properties
* With size apply fontSize,lineHeight,margin,padding,gap

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
