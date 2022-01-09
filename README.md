# better-color-tools

Better color manipulation for Sass and JavaScript/TypeScript.

Supports:

- ✅ RGB / Hex
- ✅ HSL
- ✅ [P3]

## Installing

```
npm install better-color-tools
```

## Sass

Sass has built-in [color][sass-color] functions, but they aren’t as usable as they could be. Here’s why this library exists as an alternative.

### Mix

Let’s compare this library’s mix function to Sass’ (Sass on top; better-color-tools on bottom):

<table>
  <thead>
    <th>Blend</th>
    <th>Comparison</th>
  </thead>
  <tbody>
    <tr>
      <td>red–lime</td>
      <td><img src="./.github/images/red-lime-sass.png"><img src="./.github/images/red-lime-better.png"></td>
    </tr>
    <tr>
      <td>red–yellow</td>
      <td><img src="./.github/images/red-yellow-sass.png"><img src="./.github/images/red-yellow-better.png"></td>
    </tr>
    <tr>
      <td>blue–yellow</td>
      <td><img src="./.github/images/blue-yellow-sass.png"><img src="./.github/images/blue-yellow-better.png"></td>
    </tr>
    <tr>
      <td>blue–fuchsia</td>
      <td><img src="./.github/images/blue-fuchsia-sass.png"><img src="./.github/images/blue-fuchsia-better.png"></td>
    </tr>
    <tr>
      <td>blue–lime</td>
      <td><img src="./.github/images/blue-lime-sass.png"><img src="./.github/images/blue-lime-better.png"></td>
    </tr>
  </tbody>
</table>

It may be hard to tell a difference at first, but upon closer inspection you’ll see better results with the bottom colors in each row:

- better-color-utils produces brighter, more vibrant colors when mixing complementary colors, while Sass results in [dark, muddy colors][computer-color] (compare mid tones in all examples)
- better-color-utils gives better spacing between colors while Sass inconsistently clumps certain hues together (compare blues in all examples)
- better-color-utils produces more expected colors than Sass (compare how better-color-tools passes through teal in **blue–lime** while Sass doesn’t)

#### Usage

```scss
@use 'better-color-tools' as color;

$mix: color.mix(#1a7f37, #cf222e, 0); // 100% color 1, 0% color 2
$mix: color.mix(#1a7f37, #cf222e, 0.25); // 75%, 25%
$mix: color.mix(#1a7f37, #cf222e, 0.5); // 50%, 50%
$mix: color.mix(#1a7f37, #cf222e, 0.75); // 25%, 75%
$mix: color.mix(#1a7f37, #cf222e, 1); // 0%, 100%
```

### Lighten / Darken

⚠️ Still in development. It’s important to note that Sass’ new [`color.scale()`][sass-color-scale] utility is now a fantastic way to lighten / darken colors (previous attempts had been lacking). `color.scale()` produces better results than this library,
currently, and I’m not happy with that 🙂.

```scss
@use 'better-color-tools' as color;

$lighter: color.lighten(#cf222e, 0); // 0% lighter (original color)
$lighter: color.lighten(#cf222e, 0.25); // 25% lighter
$lighter: color.lighten(#cf222e, 1); // 100% lighter (pure white)

$darker: color.darken(#cf222e, 0); // 0% darker (original color)
$darker: color.darken(#cf222e, 0.25); // 25% darker
$darker: color.darken(#cf222e, 1); // 100% darker (pure black)
```

## JavaScript / TypeScript

### Mix

[View comparison](#mix) (Sass’ mix function is a generic implementation of mixing you’ll find with other libraries in JavaScript)

_Note: you’ll see `0xcf222e` in the examples which is just another way of writing `'#cf222e'`. It’s just replacing the `#` with `0x`. Use what you prefer!_

```ts
import color from 'better-color-tools';

const mix = color.mix(0x1a7f37, 0xcf222e, 0); // 100% color 1, 0% color 2
const mix = color.mix(0x1a7f37, 0xcf222e, 0.25); // 75%, 25%
const mix = color.mix(0x1a7f37, 0xcf222e, 0.5); // 50%, 50%
const mix = color.mix(0x1a7f37, 0xcf222e, 0.75); // 25%, 75%
const mix = color.mix(0x1a7f37, 0xcf222e, 1); // 0%, 100%
```

### Lighten / Darken

⚠️ In development ([see note](#lighten--darken))

```ts
import color from 'better-color-tools';

color.lighten(0xcf222e, 0); // 0% lighter (original color)
color.lighten(0xcf222e, 0.25); // 25% lighter
color.lighten(0xcf222e, 1); // 100% lighter (pure white)

color.darken(0xcf222e, 0); // 0% darker (original color)
color.darken(0xcf222e, 0.25); // 25% darker
color.darken(0xcf222e, 1); // 100% darker (pure black)
```

### Conversion

Color conversion between RGB and hexadecimal is a trivial 1:1 conversion, so this library isn’t better than any other in that regard.

It’s in HSL handling where approaches differ. Few realize that when using whole numbers in HSL, [it only has 14.5% the colors of RGB][hsl-rgb]. In order to recreate the full RGB spectrum you need **at least 1 decimal place in all H, S, and L values.** For
this reason, any library that uses whole numbers in HSL out-of-the-box will result in distorted colors and quality loss (compare this library to [color-convert] converting from RGB -> HSL and back again):

```ts
color.from(color.from([167, 214, 65]).hsl).rgbVal; // ✅ [167, 214, 65]
convert.hsl.rgb(convert.rgb.hsl(167, 214, 65)); // ❌ [168, 215, 66]
```

The reason, again, is rounding by default. This is a [known limitation of HSL][hsl], so many libraries can disable rounding with overrides, but in addition to that not being default behavior it also produces noisy results:

```ts
color.from([167, 214, 65]).hsl; // hsl(78.93, 64.5%, 54.71%)
convert.rgb.hsl.raw([167, 214, 65]); // hsl(78.9261744966443, 64.50216450216452%, 54.70588235294118%)
```

This library takes the opinion that **HSL should have RGB precision by default.** So this library generates values that support infinite conversions without quality loss that are still readable.

#### Usage

`color.from()` takes any valid CSS string, hex number, or RGBA array as an input, and can generate any desired output as a result:

```ts
import color from 'better-color-tools';

// convert color to hex
color.from('rgb(196, 67, 43)').hex; // '#c4432b'
color.from([196, 67, 43]).hex; // '#c4432b'
color.from('rgb(196, 67, 43)').hexVal; // 0xc4432b

// convert hex to p3
color.from('rgb(196, 67, 43, 0.8)').p3; // 'color(display-p3 0.76863 0.26275 0.16863/0.8)'

// convert from p3 to hex
color.from('color(display-p3 0.23 0.872 0.918)').hex; // #3bdeea

// convert color to rgb
color.from('#C4432B').rgb; // 'rgb(196, 67, 43)'
color.from(0xc4432b).rgb; // 'rgb(196, 67, 43)'
color.from('#C4432B').rgbVal; // [196, 67, 43, 1]

// convert hex to rgba
color.from('#C4432B').rgba; // 'rgba(196, 67, 43, 1)'
color.from(0xc4432b).rgba; // 'rgba(196, 67, 43, 1)'
color.from('#C4432B80').rgbaVal; // [196, 67, 43, 0.5]

// convert color to hsl
color.from('#C4432B').hsl; // 'hsl(9.41, 64.02%, 46.86%, 1)'
color.from(0xc4432b).hsl; // 'hsl(9.41, 64.02%, 46.86%, 1)'
color.from('#C4432B').hslVal; // [9.41, 0.6402, 0.4686, 1]

// convert hsl to rgb
color.from('hsl(328, 100%, 54%)').rgb; // 'rgb(255, 20, 146)'

// convert color names to hex
color.from('rebeccapurple').hex; // '#663399'
```

#### A note on P3

The [P3 colorspace][p3] is larger than RGB. As a result, many tools apply [gamut matrices](http://endavid.com/index.php?entry=79) to convert RGB to P3 and vice-versa. While that is needed when dealing with image editing software and white-balancing, it’s
unnecessary for the web. Since browsers are better at this conversion (and may improve it over time), this library takes an intentional “hands-off” approach where P3 is equated with **Ideal RGB**, e.g.:

| P3 Color |  Ideal RGB   |  colorjs.io  |
| :------: | :----------: | :----------: |
| `1 0 0`  | ✅ `255 0 0` | ❌ `250 0 0` |

This approach produces better color conversion across-the-board by letting the browser make conversions rather than the library, which is also the approach
[recommended by Apple for CSS](https://webkit.org/blog/10042/wide-gamut-color-in-css-with-display-p3/). TL;DR this library’s P3 conversion is optimized for web.

## TODO / Roadmap

- **Planned**: Adding color spaces like [Adobe](https://en.wikipedia.org/wiki/Adobe_RGB_color_space) and [Rec 709](https://en.wikipedia.org/wiki/Rec._709) to allow color mixing and lightening/darkening to use different perceptual color algorithms

[color-convert]: https://github.com/Qix-/color-convert
[computer-color]: https://www.youtube.com/watch?v=LKnqECcg6Gw&vl=en
[hsl]: https://en.wikipedia.org/wiki/HSL_and_HSV#Disadvantages
[hsl-rgb]: https://pow.rs/blog/dont-use-hsl-for-anything/
[number-precision]: https://github.com/nefe/number-precision
[p3]: https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color()
[sass-color]: https://sass-lang.com/documentation/modules/color
[sass-color-scale]: https://sass-lang.com/documentation/modules/color#scale
