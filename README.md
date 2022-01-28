# better-color-tools

Better color manipulation for Sass and JavaScript/TypeScript. Fast (`75,000` ops/s) and lightweight (`3.7 kB` gzip).

Supports:

- ✅ RGB / Hex
- ✅ HSL
- ✅ [P3]

👉 **Playground**: https://better-color-tools.pages.dev/

## Installing

```
npm install better-color-tools
```

## Mix

Not all mixing algorithms are created equal. A proper color mixer requires [gamma correction][gamma], something most libraries omit (even including Sass, CSS, and SVG). Compare this library’s gamma-corrected results (top) with most libraries’ default mix
function:

![](./.github/images/r-g.png)

![](./.github/images/g-b.png)

![](./.github/images/b-y.png)

![](./.github/images/k-c.png)

![](./.github/images/k-w.png)

Notice all the bottom gradients have muddy/grayed-out colors in the middle as well as clumping (colors bunch up around certain shades or hues). But fear not! better-color-utils will always give you those beautiful, perfect color transitions you deserve.

```scss
// Sass
@use 'better-color-tools' as better;

$mix: better.mix(#1a7f37, #cf222e, 0); // 100% color 1, 0% color 2
$mix: better.mix(#1a7f37, #cf222e, 0.25); // 75%, 25%
$mix: better.mix(#1a7f37, #cf222e, 0.5); // 50%, 50%
$mix: better.mix(#1a7f37, #cf222e, 0.75); // 25%, 75%
$mix: better.mix(#1a7f37, #cf222e, 1); // 0%, 100%
```

```ts
// JavaScript / TypeScript
import better from 'better-color-tools';

const mix = better.mix(0x1a7f37, 0xcf222e, 0); // 100% color 1, 0% color 2
const mix = better.mix(0x1a7f37, 0xcf222e, 0.25); // 75%, 25%
const mix = better.mix(0x1a7f37, 0xcf222e, 0.5); // 50%, 50%
const mix = better.mix(0x1a7f37, 0xcf222e, 0.75); // 25%, 75%
const mix = better.mix(0x1a7f37, 0xcf222e, 1); // 0%, 100%
```

_Note: `0xcf222e` in JS is just another way of writing `'#cf222e'` (replacing the `#` with `0x`). Either are valid; use whichever you prefer!_

#### Advanced: gamma adjustment

To change the gamma adjustment, you can pass in an optional 4th parameter. The default gamma is `2.2`, but you may adjust it to achieve different results (if unsure, best to always omit this option).

```scss
// Sass
$gamma: 2.2; // default
$mix: better.mix(#1a7f37, #cf222e, 0, $gamma);
```

```ts
// JavaScript / TypeScript
const gamma = 2.2; // default
const mix = better.mix(0x1a7f37, 0xcf222e, 0, gamma);
```

## Lighten / Darken

![](./.github/images/k-c.png)

_Top: better-color-utils / Bottom: RGB averaging_

The lighten and darken methods also use [gamma correction][gamma] for improved results (also better than Sass’ `color.lighten()` and `color.darken()`). This method is _relative_, so no matter what color you start with, `darken(…, 0.5)` will always be
halfway to black, and `lighten(…, 0.5)` will always be halfway to white.

```scss
// Sass
@use 'better-color-tools' as better;

$lighter: better.lighten(#cf222e, 0); // 0% lighter (original color)
$lighter: better.lighten(#cf222e, 0.25); // 25% lighter
$lighter: better.lighten(#cf222e, 1); // 100% lighter (pure white)

$darker: better.darken(#cf222e, 0); // 0% darker (original color)
$darker: better.darken(#cf222e, 0.25); // 25% darker
$darker: better.darken(#cf222e, 1); // 100% darker (pure black)
```

```ts
// JavaScript / TypeScript
import better from 'better-color-tools';

better.lighten(0xcf222e, 0); // 0% lighter (original color)
better.lighten(0xcf222e, 0.25); // 25% lighter
better.lighten(0xcf222e, 1); // 100% lighter (pure white)

better.darken(0xcf222e, 0); // 0% darker (original color)
better.darken(0xcf222e, 0.25); // 25% darker
better.darken(0xcf222e, 1); // 100% darker (pure black)
```

## Gradient

![](./.github/images/b-g-gradient.png)

_Top: better-color-utils / Bottom: standard CSS gradient_

CSS gradients and SVG gradients are, sadly, not gamma-optimized. But you can fix that with `better.gradient()`. While there’s no _perfect_ fix for this, this solution drastically improves gradients without bloating filesize.

```scss
// Sass

// Coming soon!
```

```ts
// JavaScript/TypeScript
import better from 'better-color-tools';

const badGradient = 'linear-gradient(90deg, red, lime)';
better.gradient(badGradient); // linear-gradient(90deg,#ff0000,#e08800,#baba00,#88e000,#00ff00)
better.gradient(badGradient, true); // linear-gradient(90deg,color(display-p3 0 0 1), … )
```

`better.gradient()` takes any valid CSS gradient as its first parameter. Also specify `true` as the 2nd parameter to generate a P3 gradient instead of hex.

## Perceived Lightness

HSL’s lightness is basically worthless as it’s distorted by the RGB colorspace and has no bearing in actual color brightness or human perception. **Don’t use HSL for lightness!** Instead, use the following:

```js
import better from 'better-color-tools;

const DARK_PURPLE = '#542be9';

// lightness: get human-perceived brightness of a color (blues will appear darker than reds and yellows, e.g.)
better.lightness(DARK_PURPLE); // 0.3635 (~36% lightness)

// luminance: get absolute brightness of a color (this may not be what you want!)
better.luminance(DARK_PURPLE); // 0.0919 (~9% luminance)

// HSL (for comparison)
better.from(DARK_PURPLE).hslVal[3]; // 0.5412 (54%!? there’s no way this dark purple is that bright!)
```

## Color Formats

#### Sass

Sass already has many [built-in converters][sass-convert], so this library only extends what’s there. Here are a few helpers:

##### P3

The `p3()` function can convert any Sass-readable color into [P3][p3]:

```scss
@use 'better-color-tools' as better;

$green: #00ff00;
$blue: #0000ff;

color: $green; // #00ff00
color: better.p3($green); // color(display-p3 0 1 0)

background: linear-gradient(135deg, $green, $blue); // linear-gradient(135deg, #00ff00, #0000ff)
background: linear-gradient(135deg, better.p3($green), better.p3($blue)); // linear-gradient(135deg, color(display-p3 0 1 0), color(dipslay-p3 0 0 1)))
```

⚠️ Be sure to always include fallback colors when using P3

#### JavaScript / TypeScript

`better.from()` takes any valid CSS string, hex number, or RGBA array (values normalized to `1`) as an input, and can generate any desired output as a result:

```ts
import better from 'better-color-tools';

better.from('rgb(196, 67, 43)').hex; // '#c4432b'
better.from('rebeccapurple').hsl; // 'hsl(270, 50%, 40%)'
```

| Code                    |    Type    | Example                     |
| :---------------------- | :--------: | :-------------------------- |
| `better.from(…).hex`    |  `string`  | `"#ffffff"`                 |
| `better.from(…).hexVal` |  `number`  | `0xffffff`                  |
| `better.from(…).rgb`    |  `string`  | `"rgb(255, 255, 255)"`      |
| `better.from(…).rgbVal` | `number[]` | `[1, 1, 1, 1]`              |
| `better.from(…).rgba`   |  `string`  | `"rgba(255, 255, 255, 1)"`  |
| `better.from(…).hsl`    |  `string`  | `"hsl(360, 0%, 100%)"`      |
| `better.from(…).hslVal` | `number[]` | `[360, 0, 1, 1]"`           |
| `better.from(…).p3`     |  `string`  | `"color(display-p3 1 1 1)"` |

#### A note on HSL

[HSL is lossy when rounding to integers][hsl-rgb], so better-color-tools will yield better results than any library that rounds HSL, or rounds HSL by default.

#### A note on CSS color names

This library can convert _FROM_ a CSS color name, but can’t convert _INTO_ one (as over 99% of colors have no standardized name). However, you may import `better-color-tools/dist/css-names.js` for an easy-to-use map for your purposes.

#### A note on P3

When converting to or from P3, this library converts “lazily,” meaning the R/G/B channels are converted 1:1. This differs from some conversions which attempt to simulate hardware differences. Compare this library to colorjs.io:

| P3 Color | better-color-tools | colorjs.io |
| :------: | :----------------: | :--------: |
| `1 0 0`  |     `255 0 0`      | `250 0 0`  |

For the most part, this approach makes P3 much more usable for web and is even [recommended by Apple for Safari](https://webkit.org/blog/10042/wide-gamut-color-in-css-with-display-p3/).

## TODO / Roadmap

- **Planned**: LAB conversion
- **Planned**: Sass function for Gamma-corrected gradients

[color-convert]: https://github.com/Qix-/color-convert
[hsl]: https://en.wikipedia.org/wiki/HSL_and_HSV#Disadvantages
[hsl-rgb]: https://pow.rs/blog/dont-use-hsl-for-anything/
[gamma]: https://observablehq.com/@sebastien/srgb-rgb-gamma
[number-precision]: https://github.com/nefe/number-precision
[p3]: https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color()
[sass-color]: https://sass-lang.com/documentation/modules/color
[sass-color-scale]: https://sass-lang.com/documentation/modules/color#scale
[sass-convert]: https://sass-lang.com/documentation/values/colors
