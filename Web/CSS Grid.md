
```css
.grid-container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  gap: 10px;
}
```

`display: grid;` — enables CSS Grid layout on the container.  
`grid-template-columns: 100px 100px 100px;` — creates three columns, each 100px wide.  
`gap: 10px;` — sets 10px spacing between rows and columns.

---

```css
.example {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
}
```

Creates three columns: first and third take equal space, middle column takes double that space.

---

```css
.example {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
}
```

Creates four equal-width columns, each taking up one fraction of available space.

---

```css
.example {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
}
```

Creates a responsive grid where columns automatically fit, each at least 150px wide and able to expand to fill available space.

---

```css
.item1 {
  grid-column: 1 / 3;
  grid-row: 2 / 3;
  background: #f6973f;
}
```

`.item1` spans two columns (from line 1 to 3) and sits in the second row with an orange background.

---

```css
.item5 {
  grid-column: 2 / 3;
  grid-row: 1 / 2;
  background: #c3f806;
}
```

`.item5` occupies the second column in the first row with a bright green background.

---

```css
.example1 {
  justify-items: center;
  align-items: end;
}
```

Horizontally centers items in each grid cell, vertically aligns them to the bottom.

---

```css
.example2 {
  justify-items: start;
  align-items: start;
}
```

Horizontally aligns items to the left, vertically aligns them to the top.

---

```css
.example3 {
  justify-items: end;
  align-items: center;
}
```

Horizontally aligns items to the right, vertically centers them.

---

```css
.example4 {
  justify-items: stretch;
  align-items: stretch;
}
```

Items stretch to fill the entire width and height of their grid cells (default).

---

```css
.example {
  place-items: center;
}
```

Shorthand for `align-items` and `justify-items`; centers items both vertically and horizontally in their grid cells.

---

```css
.example1 {
  display: grid;
  align-content: start;
  justify-content: start;
}
```

Packs rows and columns toward the start (top-left).

---

```css
.example2 {
  display: grid;
  align-content: end;
  justify-content: end;
}
```

Packs rows and columns toward the end (bottom-right).

---

```css
.example3 {
  display: grid;
  align-content: center;
  justify-content: center;
}
```

Centers rows and columns in the grid container.

---

```css
.example4 {
  display: grid;
  align-content: space-between;
  justify-content: space-between;
}
```

Distributes rows and columns with even gaps **between** them.

---

```css
.example5 {
  display: grid;
  align-content: space-around;
  justify-content: space-around;
}
```

Adds equal space **around** rows and columns (half-size space at edges).

---

```css
.example6 {
  display: grid;
  align-content: space-evenly;
  justify-content: space-evenly;
}
```

Gives rows and columns equal spacing **between and at edges**.

---

```css
.example {
  grid-auto-rows: 60px;
}
```

Automatically creates rows of 60px height for items placed outside defined rows.

---

```css
.example {
  grid-auto-columns: 60px;
}
```

Automatically creates columns of 60px width for items placed outside defined columns.

---

```css
.example1 {
  display: grid;
  grid-auto-flow: row;
}
```

Places items row by row (default), wrapping to the next row when full.

---

```css
.example2 {
  display: grid;
  grid-auto-flow: column;
}
```

Places items column by column, wrapping to the next column when full.

---

```css
.example3 {
  display: grid;
  grid-auto-flow: row dense;
}
```

Places items row by row, trying to back-fill gaps for a tighter layout.

---

```css
.example4 {
  display: grid;
  grid-auto-flow: column dense;
}
```

Places items column by column, filling gaps densely.

---
```css
.container {
  display: grid;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
}
```

`grid-template-areas` defines **named regions** in the grid using a visual map of strings.

- Each quoted string represents a **row** in the grid.
    
- Inside a row, each name represents a **cell** in that row.
    
- Repeating the same name makes an element span multiple columns or rows.
    
- A dot (`.`) can be used to create an empty cell with no assigned area.
    

Example explained:

- `"header header"` → `header` spans across **two columns** in the first row.
    
- `"sidebar main"` → second row has `sidebar` in first column, `main` in second column.
    
- `"footer footer"` → `footer` spans across both columns in the last row.
    

Then, individual items are assigned with `grid-area`:

```css
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

Each element is placed into its named slot, making layouts easier to read and maintain.

---
