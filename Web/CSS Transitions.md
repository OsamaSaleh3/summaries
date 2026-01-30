```css
/* transition-property: defines which CSS properties animate */
.example1 {
  transition-property: width;
}
```

Animates only the `width` property.

---

```css
/* transition-duration: sets how long the animation lasts */
.example2 {
  transition-duration: 2s;
}
```

Animation lasts for 2 seconds.

---

```css
/* transition-delay: sets wait time before animation starts */
.example3 {
  transition-delay: 0.5s;
}
```

Animation starts 0.5 seconds after the property changes.

---

```css
/* transition-timing-function: controls acceleration & deceleration */
.example4 {
  transition-timing-function: ease-in-out;
}
```

Animation eases in and out smoothly.

---

```css
/* transition: shorthand for property, duration, timing, delay */
.example5 {
  transition: width 2s ease 0.5s;
}
```

Animates `width` over 2s with ease, starting after 0.5s delay.

---

