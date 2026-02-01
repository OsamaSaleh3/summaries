
```html
<span>Inline text</span>
```

`span` is an inline element; it only takes the width its content needs and doesn’t break the line.

---

```html
<div>Block element</div>
```

`div` is a block element; it expands to the full line width and starts on a new line.

---

```html
<p>Use <b>bold</b> or <strong>important</strong> text.</p>
```

Both render bold, but `strong` conveys semantic importance for assistive technologies.

---

```html
<p><i>Italic style</i> and <em>emphasized meaning</em></p>
```

Both render italic, but `em` adds semantic emphasis (a weaker level than `strong`) for screen readers.

---

```html
<p><u>Underlined text</u></p>
```

The `u` tag underlines text.

---

```html
<p><small>Footnote or less-important text</small></p>
```

`small` reduces font size and semantically marks side comments/less important content.

---

```html
<p>Find the <mark>highlighted</mark> part.</p>
```

`mark` highlights text to indicate relevance.

---

```html
<p>Old: <del>Blue widget</del> New: <ins>Green widget</ins></p>
```

`del` shows removed text (strikethrough) and `ins` shows added text (usually underlined), both with semantic change meaning.

---

```html
<img src="image.jpg" title="Scenic lake" style="width:25%; height:auto;" alt="Scenic lake" />
```

`title` shows on hover; `width:25%` with `height:auto` keeps the aspect ratio while sizing.

---

```html
<img src="a.jpg" alt="A" />
<img src="b.jpg" alt="B" />
```

Browsers fetch images asynchronously and can download them concurrently while rendering the page.

---

```html
<img src="a.jpg" alt="A" />
<img src="b.jpg" alt="B" />
```

Each `<img>` generally triggers its own HTTP request (unless cached/embedded).

---

```html
<img src="photo.jpg" width="1200" height="800" style="height:auto;" alt="Photo with reserved space" />
```

Setting intrinsic `width` and `height` reserves layout space to prevent content shifting (CLS).

---

```html
<picture>
  <source srcset="photo.avif" type="image/avif" />
  <source srcset="photo.webp" type="image/webp" />
  <img src="photo.jpg" width="1200" height="800" alt="Optimized photo" />
</picture>
```

Use appropriately sized images and efficient formats (AVIF/WebP often smaller than JPEG; PNG is larger for photos).

---

```html
<img src="https://cdn.example.com/images/photo.jpg" alt="From CDN" />
```

Serving assets from a CDN delivers files from edge servers closer to users for faster loads.

---

```html
<img src="gallery-3.jpg" loading="lazy" alt="Below-the-fold image" />
```

Use lazy loading for images that appear below the fold; avoid it for critical, above-the-fold images.

---

```html
<img src="gallery-3.jpg" loading="auto" alt="Browser decides" />
```

`loading="auto"` (default) lets the browser decide whether to load lazily or eagerly.

---

```html
<img src="hero.jpg" loading="eager" alt="Hero image" />
```

`loading="eager"` tells the browser to load the image immediately—ideal for above-the-fold/critical images.

---

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

Defines the viewport to match device width and sets initial zoom to 1, enabling responsive design.

---

```html
<img
  src="image-600.jpg"
  srcset="image-600.jpg 600w, image-900.jpg 900w, image-1500.jpg 1500w"
  sizes="(max-width: 700px) 90vw, 700px"
  alt="A scenic view"
/>
```

Provides multiple image files with declared widths; the browser chooses the best file based on viewport and the `sizes` hint.

---

```html
<a>Click here</a>
```

An anchor without `href` behaves like a generic inline container and is not a clickable link.

---

```html
<a href="https://example.com">Visit site</a>
```

`href` sets the link destination (URL or fragment).

---

```html
<a href="https://example.com" title="Opens the example site">Example</a>
```

`title` adds a tooltip on hover; it does not replace accessible labels.

---

```html
<a href="https://example.com" target="_parent">Open in parent</a>
```

`target="_parent"` opens in the parent browsing context (useful in iframes).

---

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">Open in new tab</a>
```

`target="_blank"` opens a new tab/window; add `rel="noopener noreferrer"` for security/privacy.

---

```html
<a href="https://example.com" target="_top">Open in top</a>
```

`target="_top"` loads the URL in the topmost frame, breaking out of nested frames.

---

```html
<a href="https://example.com" target="_self">Open here</a>
```

`target="_self"` opens in the same tab (default behavior).

---

```html
<a href="mailto:support@example.com?subject=Help&body=Hi">Email us</a>
```

`mailto:` opens the user’s email client with prefilled fields.

---

```html
<a href="tel:+1234567890">Call us</a>
```

`tel:` initiates a phone call on supported devices.

---

```html
<a href="/files/report.pdf" download="Q3-report.pdf">Download report</a>
```

`download` prompts saving the linked file (optionally with a suggested filename).

---

```html
<a href="#features">Go to Features</a>
<section id="features">...</section>
```

An internal “bookmark” link uses a fragment `#id` to jump to a page section.

---

```html
<a href="https://example.com/product" ping="https://analytics.example.com/ping https://log.example.com/click">Product</a>
```

`ping` sends POST requests to the listed URLs when the link is followed (click tracking).

---

```html
<a href="https://example.com" rel="noopener noreferrer nofollow">External link</a>
```

`rel` defines the link relationship; common values: `noopener`/`noreferrer` for security/privacy and `nofollow` for SEO hints.

---

```html
<picture>
  <source media="(min-width: 800px)" srcset="large.jpg" />
  <source media="(min-width: 450px)" srcset="medium.jpg" />
  <img src="small.jpg" alt="An example image" />
</picture>
```

Chooses a large image at ≥800px, a medium image at ≥450px, and falls back to the small image otherwise.

---

```html
<embed src="assets/A2.jpg" type="image/jpeg" width="600" height="400">
```

Embeds a JPEG image at fixed dimensions; use `<img>` for standard images.

---

```html
<embed src="assets/Test.pdf" type="application/pdf" width="600" height="500">
```

Displays a PDF inline with a built-in viewer at the given size.

---

```html
<embed src="assets/Funny.mp4" type="video/mp4" width="600" height="400">
```

Embeds an MP4 video; prefer `<video controls>` for native, accessible playback.

---

```html
<embed src="assets/sample.mp3" type="audio/mpeg" width="300" height="250">
```

Embeds an MP3 audio resource; prefer `<audio controls>` for playback and accessibility.

---

```html
<embed src="assets/OtherHTMLPage.html" type="text/html" width="600" height="400">
```

Embeds another HTML page inline (akin to an `<iframe>`).

---

## Semantic Elements

```html
<header>Site Title</header>
<nav>...</nav>
<main>...</main>
<aside>Sidebar</aside>
<footer>© 2025</footer>
```

Core semantic layout elements that convey meaning to browsers and assistive technologies beyond mere presentation.

---

```html
<section aria-labelledby="intro-title">
  <h1 id="intro-title">Understanding HTML Semantic Elements</h1>
  <p>Use meaningful tags to structure content for accessibility, SEO, and maintainability.</p>
</section>
```

Demonstrates using semantic tags and headings to communicate document structure to users and machines.

---

```html
<header>
  <h1>Blog Name</h1>
  <p>Tagline or intro content</p>
</header>
```

`header` contains introductory content or navigational aids for a page or section.

---

```html
<footer>
  <small>© 2025 Example Co. · <a href="/privacy">Privacy</a></small>
</footer>
```

`footer` holds closing content like copyright, metadata, and related links.

---

```html
<nav aria-label="Primary">
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>
```

`nav` represents a section of navigation links and serves as a landmark for assistive tech.

---

```html
<main id="content">
  <h1>Article Title</h1>
  <p>Main content goes here.</p>
</main>
```

`main` encloses the unique primary content of the page (only one per page).

---

```html
<section aria-labelledby="features-title">
  <h2 id="features-title">Features</h2>
  <p>Grouped related content with its own heading.</p>
</section>
```

`section` groups thematically related content and should typically include a heading.

---

```html
<article>
  <h2>News Story</h2>
  <p>Self-contained content that could stand alone or be syndicated.</p>
</article>
```

`article` is independent, self-contained content like a post, story, or comment.

---

```html
<aside>
  <h2>Related</h2>
  <p>Supplementary info like side notes or promos.</p>
</aside>
```

`aside` contains tangential or complementary content to the main flow.

---

```html
<p>Remember the <mark>submission deadline</mark> is Friday.</p>
```

`mark` highlights text to denote relevance or a match within a context.

---

```html
<figure>
  <img src="mountain.jpg" alt="Snowy mountain at sunrise" />
  <figcaption>Figure 1: Sunrise over the ridge.</figcaption>
</figure>
```

`figure` groups self-contained media with an optional `figcaption` describing it.

---

```html
<details>
  <summary>Show more details</summary>
  <p>Hidden content revealed when expanded.</p>
</details>
```

`details` creates a disclosure widget with a `summary` label that toggles its contents.

---

```html
<p>Event starts at <time datetime="2025-09-01T09:00">September 1, 2025, 9:00 AM</time>.</p>
```

`time` represents a date or time with a machine-readable `datetime` value.

---

```html
<!-- Use semantic elements to clarify meaning, improve accessibility, and aid SEO -->
```

Semantic markup improves structure, accessibility, reuse, and discoverability across tools and devices.

---
## Blockquotes, Quotes, and Citations: Used to mark up quotations and their sources — `blockquote` for long excerpts, `q` for short inline quotes, and `cite` for citing works


```html
<blockquote cite="https://example.com/article">
  <p>The web does not just connect machines; it connects people.</p>
</blockquote>
```

A block-level quotation for longer excerpts; the optional `cite` attribute can point to the source URL or identifier.

---

```html
<p>— Tim Berners-Lee, <cite><a href="https://example.com/weaving-the-web">Weaving the Web</a></cite></p>
```

Marks the title of a work (book, article, film, etc.); not for people’s names. Often paired with a link to the source.

---

```html
<p>She said, <q cite="https://example.com/interview">The web is for everyone</q>.</p>
```

An inline quotation for short quoted text; browsers typically add quotation marks, and `cite` can reference the source.

---

## Specialized Text-Level Semantic Elements: Provide meaning for specific inline text — `dfn` for definitions, `abbr` for abbreviations, `code` for code snippets, `samp` for program output, `kbd` for user input, and `var` for variables.


```html
<p><dfn id="api">API</dfn> — Application Programming Interface</p>
```

Marks the term being defined so the document semantically associates this occurrence with its definition.

---

```html
<p><abbr title="Cascading Style Sheets">CSS</abbr> controls visual styling.</p>
```

Represents an abbreviation or acronym; the `title` provides the full expansion for tooltips and assistive tech.

---

```html
<p>Run <code>npm install</code> to install dependencies.</p>
```

Denotes snippets of computer code or commands; use `<pre><code>` for multi-line blocks.

---

```html
<p>Program output: <samp>File not found</samp></p>
```

Represents sample output from a program, system, or device.

---

```html
<p>Save the file with <kbd>Ctrl</kbd> + <kbd>S</kbd>.</p>
```

Indicates user input via keyboard (or similar input), often rendered in a monospace style.

---

```html
<p>Area = π × <var>r</var>²</p>
```

Represents a variable or placeholder name in mathematics or programming.

---

```html
<label>Progress: <meter value="65" min="0" max="100" low="25" high="80" optimum="70">65%</meter></label>
```

Displays a gauge for a value within a known range, with optional thresholds and an optimum.

---

```html
<link rel="preload" href="/fonts/Inter.var.woff2" as="font" type="font/woff2" crossorigin>
```

Hints the browser to fetch a critical resource early (specify `as` to match the resource type).

---

