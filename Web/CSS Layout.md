```css
.example1 {
  position: relative;
  z-index: 1;
}
```

Places the element **above** elements with lower or default `z-index`.

---

```css
.example2 {
  position: relative;
  z-index: 10;
}
```

Higher values bring the element further **to the front** of the stacking order.

---

```css
.example3 {
  position: relative;
  z-index: -1;
}
```

Negative values push the element **behind** others.

---

**Note:** `z-index` only works on elements with a positioning (`relative`, `absolute`, `fixed`, or `sticky`).

---

```css
.example1 {
  overflow-y: hidden;
}
```

Clips content vertically; no vertical scroll allowed.

---

```css
.example2 {
  overflow-x: auto;
}
```

Adds horizontal scrollbar only if content overflows.

---

```css
.example3 {
  overflow: visible;
}
```

Default — content can spill outside the box. 🏞️

---

```css
.example4 {
  overflow: hidden;
}
```

Clips overflowing content; hidden from view. ✂️

---

```css
.example5 {
  overflow: scroll;
}
```

Always shows scrollbars, even if not needed. 📜

---

```css
.example6 {
  overflow: auto;
}
```

Scrollbars appear only when content overflows. 🔄

---

```css
.example7 {
  overflow: clip;
}
```

Clips overflowing content and prevents programmatic scrolling. 🚫

---

```css
.img-left {
  float: left;
}
```

Element floats to the left, and other inline content wraps around it.

---

```css
.img-right {
  float: right;
}
```

Element floats to the right, with text/content wrapping around it.

---

```css
.no-float {
  float: none;
}
```

Removes float; element stays in normal flow.

---

```css
.clear-left {
  clear: left;
}
```

Element moves below any left-floated elements.

---

```css
.clear-right {
  clear: right;
}
}
```

Element moves below any right-floated elements.

---

```css
.clear-both {
  clear: both;
}
```

Element moves below both left- and right-floated elements.

---


