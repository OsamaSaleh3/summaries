
```css
@media screen and (max-width: 600px) {
  .example {
    background: lightblue;
  }
}
```

Applies styles only when the device is a screen and viewport width is 600px or less.

---

```css
@media screen and (min-width: 1200px) {
  .box {
    flex: 1 1 calc(25% - 1rem);
    background: #b8daff;
  }
}
```

When the screen width is **1200px or larger**, each `.box` flex item takes up about 25% minus 1rem spacing and gets a light blue background.

---

```css
@media screen and (orientation: portrait) {
  body {
    background: lightcoral;
  }
}
```

Applies a light coral background to the page when the device is in **portrait orientation**.

---

```css
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .hero {
    background-image: url('Images/bg2x.jpg');
  }
}
```

Loads a higher-resolution background image for `.hero` on devices with **Retina/HiDPI displays**.

---

```css
@media print {
  body {
    font-size: 12pt;
    color: #000;
  }
  nav,
  .no-print {
    display: none;
  }
}
```

Formats page for printing: sets readable font size and black text, hides navigation and `.no-print` elements.

---

