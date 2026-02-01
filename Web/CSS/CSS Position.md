```css
.example {
  position: static;
  /* no positioning attributes apply (top/left/etc. are ignored) */
}
```

`static` cannot be offset; element just follows normal flow.

---

```css
.example {
  position: relative;
  top: 20px;
  left: 10px;
}
```

`relative` can be shifted using `top`, `right`, `bottom`, `left` but still occupies its original space.

---

```css
.example {
  position: absolute;
  top: 50px;
  right: 20px;
}
```

`absolute` uses `top`, `right`, `bottom`, `left` relative to the nearest positioned ancestor.

---

```css
.example {
  position: fixed;
  bottom: 10px;
  right: 10px;
}
```

`fixed` uses `top`, `right`, `bottom`, `left` relative to the viewport and stays on screen when scrolling.

---

```css
.example {
  position: sticky;
  top: 0;
}
```

`sticky` combines normal flow with fixed behavior; controlled by `top`, `right`, `bottom`, or `left`.

---
