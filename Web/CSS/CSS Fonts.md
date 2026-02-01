```css
.example {
  font-family: font1, font2, generic-family;
}
```

Defines prioritized fonts: tries `font1` first, then `font2`, and falls back to `generic-family` (like serif, sans-serif, monospace).

---

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Roboto&display=swap" rel="stylesheet">

<style>
.example {
  font-family: 'Roboto', sans-serif;
}
</style>
```

Adds `preconnect` for performance, loads Google Fonts via `<link>`, and applies the font in CSS with fallbacks.

---

```css
@font-face {
  font-family: "YourFontName";       /* Internal reference name */
  src:
    url("fonts/YourFont.woff2") format("woff2"), /* Preferred format */
    url("fonts/YourFont.woff")  format("woff");  /* Fallback format */
  font-weight: 400;                  /* Normal weight */
  font-style: normal;                /* Normal, italic, etc. */
  font-display: swap;                /* Controls text rendering strategy */
  unicode-range: U+000-5FF;          /* Optional: define character subset */
}
```

Defines a custom font with multiple sources for compatibility, sets its weight and style, controls loading behavior, and optionally restricts character ranges.

---
```css
selector {
  font: [ font-style? font-variant? font-weight? ]?
        font-size[/line-height]?
        font-family;
}
```

Shorthand for font properties: order is style → variant → weight → size (with optional line-height) → family (required).

---


