```css
.tooltip {
  position: relative;
  display: inline-block;
  cursor: pointer;
}
```

The tooltip container; inline-block with relative positioning so child tooltip text can be placed absolutely.

---

```css
.tooltip .tooltip-text {
  visibility: hidden;
  position: absolute;
  bottom: 125%;
  left: 50%;
  transform: translateX(-50%);
  background: #333;
  color: #fff;
  padding: 0.5rem;
  border-radius: 4px;
  white-space: nowrap;
  font-size: 0.9rem;
  opacity: 0;
  transition: opacity 0.2s;
  pointer-events: none;
}
```

The tooltip text is absolutely positioned above the element, styled with a dark background, white text, padding, and rounded corners. It starts hidden (`visibility: hidden; opacity: 0`) and ignores pointer events.

---

```css
.tooltip:hover .tooltip-text {
  visibility: visible;
  opacity: 1;
}
```

When hovering the tooltip container, the tooltip text fades in and becomes visible.

---

