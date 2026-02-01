
## Social Sharing (Open Graph & Twitter)


```html
<meta charset="utf-8">
```

Declares the document’s character encoding as UTF-8.

---

```html
<meta name="language" content="English">
```

Indicates the primary language of the page content.

---

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

Controls mobile layout: matches device width and sets initial zoom to 1.

---

```html
<meta name="description" content="An overview of HTML metadata elements and their uses.">
```

Provides a concise summary of the page for search engines and previews.

---

```html
<meta name="keywords" content="HTML, metadata, SEO, web development">
```

Lists topic keywords associated with the page.

---

```html
<meta name="author" content="Jane Doe">
```

Identifies the author of the document.

---

```html
<meta name="revised" content="Saturday, September 7th, 2025, 5:15 pm">
```

Notes the last modified date and time of the page.

---

```html
<meta name="copyright" content="Copyright 2022 Jane Doe">
```

States the copyright ownership for the content.

---

```html
<meta name="robots" content="noindex, nofollow">
```

Tells crawlers not to index the page or follow its links.

---

```html
<meta name="generator" content="WordPress 5.9">
```

Indicates the software used to generate the page.

---

```html
<meta name="revisit-after" content="7 days">
```

Suggests how often crawlers should revisit the page.

---

```html
<meta name="rating" content="general">
```

Declares an age-appropriateness rating for the content.

---

```html
<meta name="application-name" content="HTML Metadata Guide">
```

Names the web application as shown by some browsers/OS UIs.

---

```html
<meta name="theme-color" content="#5a67d8">
```

Hints a UI theme color for the browser’s address bar and UI.

---

## HTTP-Equiv Meta Directive


```html
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
```

Sets the page MIME type and character encoding; legacy—use `<meta charset="UTF-8">` in HTML5.

---

```html
<meta http-equiv="refresh" content="5">
```

Reloads the current page every 5 seconds.

---

```html
<meta http-equiv="refresh" content="10; url=http://www.ProgrammingAdvices.com/">
```

Redirects to the given URL after 10 seconds.

---

```html
<meta http-equiv="cache-control" content="no-cache">
```

Asks the browser to revalidate before using a cached copy (don’t use cache without checking the server).

---

```html
<meta http-equiv="Pragma" content="no-cache">
```

Legacy HTTP/1.0 no-cache directive; kept for older caches.

---

```html
<meta http-equiv="expires" content="Wed, 21 Oct 2025 07:28:00 GMT">
```

Sets a specific UTC/GMT time when the cached page should be considered expired.

---

```html
<meta http-equiv="Set-Cookie" content="name=value; expires=Wed, 21 Oct 2025 07:28:00 GMT; path=/;">
```

Attempts to set a cookie from HTML; non-standard/mostly unsupported—set cookies via HTTP headers or `document.cookie`.

---

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

Forces Internet Explorer to use its latest rendering engine (legacy IE only).

---

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self';">
```

Applies a CSP that restricts all resource loads to the same origin, improving security.

---

## Robots & Crawler Directives


```html
<meta name="robots" content="index, follow">
```

Allows indexing of this page and following its links.

---

```html
<meta name="robots" content="noarchive">
```

Prevents search engines from storing a cached copy of the page.

---

```html
<meta name="robots" content="noindex">
```

Blocks the page from being indexed (won’t appear in search results).

---

```html
<meta name="robots" content="nofollow">
```

Tells crawlers not to follow any links on the page.

---

```html
<meta name="revisit-after" content="7 days">
```

Advises crawlers how often to revisit the page (largely ignored by major engines).

---

```html
<meta name="robots" content="nosnippet">
```

Prevents search results from showing text or rich snippets for this page.

---

```html
<meta name="robots" content="noimageindex">
```

Prevents images on the page from being indexed in image search.

---

```html
<meta name="googlebot" content="index, follow, max-snippet:-1">
```

Google-specific: allow indexing/following and permit unlimited snippet length.

---

```html
<meta name="bingbot" content="index, follow, noarchive">
```

Bing-specific: allow indexing/following but disallow cached copies.

---

## Social Sharing (Open Graph & Twitter)


```html
<meta property="og:title" content="Understanding Social Media Meta Tags">
```

Sets the title shown in social media share previews.

---

```html
<meta property="og:type" content="article">
```

Declares the content type as an article, influencing how platforms render the preview.

---

```html
<meta property="og:url" content="https://www.example.com/article">
```

Specifies the canonical URL so shares consolidate to one address.

---

```html
<meta property="og:image" content="https://www.example.com/thumbnail.jpg">
```

Provides the image URL used as the thumbnail in share previews.

---

```html
<meta property="og:description" content="A detailed guide on using social media meta tags to enhance content visibility.">
```

Supplies the description text shown beneath the title in previews.

---

```html
<meta name="twitter:card" content="summary_large_image">
```

Selects the large-image Twitter Card layout for richer previews.

---

```html
<meta name="twitter:creator" content="@exampleuser">
```

Attributes the content to a specific Twitter handle.

---

```html
<meta name="twitter:image:alt" content="An overview of social media optimization techniques">
```

Provides alternative text for the shared image to improve accessibility.

---


