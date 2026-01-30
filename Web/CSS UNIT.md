
```css
.example {
  margin: 20px;
}
```

`px` is an absolute unit representing pixels on the screen, independent of parent or root font size.

---
```css
.example {
  width: 2in;
}
```

`in` sets size in inches; here the element’s width is fixed at 2 physical inches on the screen or print.

---
```css
.example {
  height: 5cm;
}
```

`cm` sets size in centimeters; here the element’s height is fixed at 5 cm.

---

```css
.example {
  width: 80%;
}
```

`%` is relative to the parent element’s size; here the width is 80% of the parent’s width.

---



```css
.example {
  font-size: 2em;
}
```

`em` unit is relative to the font-size of the element’s parent; here it makes the text twice as large as the parent’s font size.

---

```css
.example {
  font-size: 1.5rem;
}
```

`rem` unit is relative to the font-size of the root element (`<html>`), making text scale consistently across the document.

---


```css
.example {
  height: 1vh;
  width: 1vw;
}
```

Sets height to 1% of viewport height and width to 1% of viewport width.

---

```css
.example {
  width: 10vmin;
  height: 10vmax;
}
```

`vmin` uses the smaller of viewport’s width or height as unit, `vmax` uses the larger.

---

```css
.example {
  width: 40ch;
}
```

Sets width relative to the width of the "0" character in the element’s font (here, ~40 characters wide).

---
```css
.example {
  line-height: 2ex;
}
```

`ex` unit equals the height of the lowercase "x" in the element’s font, useful for font-relative sizing.

---


