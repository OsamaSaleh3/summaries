```css
.example {
  box-sizing: border-box | content-box;
}
```

Element’s total width/height **includes** content, padding, and border.

---

```css
.example {
  width: 200px;
}
```

Sets total element width to 200px (padding + border are included if `border-box` is used).

---

```css
.example {
  padding: 20px;
}
```

Adds 20px spacing inside the element’s box, reducing content area if width is fixed.

---

```css
.example {
  border: 5px solid #333;
}
```

Adds a solid border of 5px thickness in dark gray color.

---

```css
.example {
  margin: 20px;
}
```

Adds 20px space outside the element, separating it from neighbors.

---

```css
.example {
  background: #ffecb3;
}
```

Fills the element’s background with a light yellow color.

---

**Difference:**

- `box-sizing: content-box` (default): `width` applies **only** to content, padding and border are added on top.
    
- `box-sizing: border-box`: `width` applies to the **entire box** (content + padding + border), so the visible size stays consistent.
    

---
