```css
.example {
  display: flex;
}
```

Creates a block-level flex container.

---

```css
.example {
  display: inline-flex;
}
```

Creates an inline-level flex container.

---

```css
.example {
  flex-direction: row;
}
```

Arranges flex items in a horizontal row (default).

---

```css
.example {
  flex-direction: row-reverse;
}
```

Arranges flex items in a horizontal row, reversed.

---

```css
.example {
  flex-direction: column;
}
```

Arranges flex items in a vertical column.

---

```css
.example {
  flex-direction: column-reverse;
}
```

Arranges flex items in a vertical column, reversed.

---

```css
.example {
  justify-content: flex-start;
}
```

Aligns items at the start of the main axis.

---

```css
.example {
  justify-content: center;
}
```

Centers items along the main axis.

---

```css
.example {
  justify-content: space-between;
}
```

Distributes items with space between them.

---

```css
.example {
  align-items: stretch;
}
```

Stretches items to fill the cross axis.

---

```css
.example {
  align-items: center;
}
```

Centers items along the cross axis.

---

```css
.example {
  align-items: flex-end;
}
```

Aligns items to the end of the cross axis.

---

```css
.example {
  flex-wrap: nowrap;
}
```

Prevents items from wrapping.

---

```css
.example {
  flex-wrap: wrap;
}
```

Allows items to wrap onto multiple lines.

---

```css
.example {
  flex-wrap: wrap-reverse;
}
```

Allows items to wrap in reverse order.

---

```css
.item {
  flex: 1 0 100px;
}
```

Item grows to fill space, doesn’t shrink, and starts with 100px base size.

---

```css
.example {
  gap: 20px;
}
```

Sets a 20px gap between rows and columns.

---

```css
.example {
  row-gap: 15px;
}
```

Sets vertical spacing between rows.

---

```css
.example {
  column-gap: 10px;
}
```

Sets horizontal spacing between columns.

---

```css
.item {
  flex: 0 0 calc(50% - 20px);
}
```

Item takes up half the container’s width minus 20px, useful for equal columns with gaps.

---

```css
.item {
  flex-basis: calc(25% - 10px);
}
```

Sets the base size of the flex item to one quarter of the container width minus 10px.

---

```css
.item {
  width: calc(100% - 50px);
}
```

Defines item width using calculation, here full width minus 50px.

---