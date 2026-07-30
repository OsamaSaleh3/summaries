
## What Is a Server, Really?

In the context of Node.js, a server is just a program that **listens for incoming network requests** and **returns a response**. That's the whole idea, stripped down.

Every time your browser loads a page, it's making network requests — you can literally watch this happen in your browser's DevTools under the "Network" tab. Each request goes out to some address on the internet, and something on the other end (a server) sends something back — HTML, JSON, images, whatever.

There are different kinds of servers:

- **Traditional servers** that run constantly, always listening.
- **CDNs** — servers replicated across the world, routing traffic to whichever copy is geographically closest, often caching content.
- **Serverless** — servers that are "off" until a request comes in, then spin up, handle it, and shut back down.

Some servers support **real-time, two-way communication** (think multiplayer games, or seeing someone else's cursor live in a Google Doc). Others, like plain HTTP, are **not real-time** — you make one request, you get one response, and that's it. Node.js can do both, but this section focuses on the simple, non-real-time case: a basic HTTP server.

## Building a Basic HTTP Server

Node ships with a built-in core module called `http` that lets you create a server without installing anything extra:

```javascript
import http from 'node:http';

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello there');
});

server.listen(4000, () => {
  console.log('Server running on http://localhost:4000');
});
```

A few things worth unpacking here:

- `http.createServer(callback)` takes a callback because handling a request is asynchronous. The callback always receives two arguments, in this order: the **request object** (`req` — "who's asking, and what did they ask for") and the **response object** (`res` — "how we reply"). You'll see this `req, res` pairing constantly in Node/Express-style code.
- **Status codes** are a simple system for telling the _other_ application (often a browser) how a request went, without it having to read and interpret the actual response body:
    
    - `200–299` → success
    - `300–399` → success, but cached (data isn't fresh)
    - `400–499` → the client/user sent a bad request (e.g. `404` = not found, `401` = unauthorized)
    - `500–599` → the server itself messed up
    
    Browsers use these to decide things like whether to show a red error in the console. Not every technology cares about them equally — GraphQL, for example, almost always returns `200` and instead encodes success/failure inside the response data itself.
- **Headers** are just key-value pairs attached to a request or response. One of the most important ones is `Content-Type`, which tells the client what _kind_ of data is being sent back, using a **MIME type** (basically the internet's version of a file extension) — e.g. `text/plain`, `text/html`, `application/json`. This is how a browser knows whether to render something as a webpage, execute it as JavaScript, display an image, etc.
- `res.end(data)` finishes the response — it's the "okay, I'm done, here's my reply" signal.
- `server.listen(port, callback)` starts the server listening on a **port** — a number your server binds to on your machine (or the network) so other software knows exactly where to send requests. Common local dev choices are things like `3000`, `4000`, `5000` — the exact number doesn't matter, though you might hit a "port already in use" error if something else is already listening on it. (Every website you visit is also on a port — usually `80` — you just don't see it because browsers hide the default port.)
- Unlike scripts you've run so far, **a server doesn't exit when it's done** — it keeps running indefinitely, waiting for the next request, until you manually stop it (e.g. `Ctrl+C` in the terminal). That's the whole point: it needs to always be available to handle whatever comes in.

## Interpolating Data into HTML (server-rendering notes)

The next goal: instead of only viewing notes through CLI commands, render them as a real webpage by injecting the notes data into an HTML template — this is essentially a simple, manual version of what React (or any templating engine) does under the hood.

**Interpolation** just means: take a string with a placeholder in it, and swap that placeholder out for a real value. For example, `"this is my name: {{ name }}"` becomes `"this is my name: Alex"` once you interpolate the `name` value into it.

**Step 1 — the HTML template with a placeholder:**

```html
<!DOCTYPE html>
<html>
  <body>
    <div class="notes">{{ notes }}</div>
  </body>
</html>
```

**Step 2 — the `interpolate` helper**, which finds any `{{ key }}`-style placeholder in a string and replaces it with the matching property from a data object:

```javascript
function interpolate(html, data) {
  return html.replace(/{{\s*(\w+)\s*}}/g, (match, key) => {
    return data[key] || '';
  });
}
```

This regex just says: look for two open curly braces, optional whitespace, a word (the key name), optional whitespace, two closing curly braces — and do this for every match in the string (`g` flag = global). Whatever key it captures, it looks up on the `data` object and swaps in that value (or an empty string if nothing matches).

**Step 3 — turning note objects into actual HTML**, similar to how you'd `.map()` over data in React, Vue, or Svelte to produce markup:

```javascript
function formatNotes(notes) {
  return notes.map(note => `
    <div class="note">
      <p>${note.content}</p>
      <div class="tags">
        ${note.tags.map(tag => `<span class="tag">${tag}</span>`).join('')}
      </div>
    </div>
  `).join('\n');
}
```

Notice the _nested_ interpolation: for each `note`, you build a `div`, and inside that, you `.map()` again over `note.tags` to build a `<span>` per tag. Both `.map()` calls need `.join('')` afterward — otherwise you'd get back a literal JavaScript array (with commas) instead of one clean HTML string.

## Wiring It All Together: the Actual Server

Now the pieces come together into a server that reads the HTML template file, interpolates the real notes into it, and sends the result back to the browser:

```javascript
const createServer = (notes) => {
  return http.createServer(async (req, res) => {
    const HTML_PATH = new URL('./template.html', import.meta.url).pathname;
    const template = await fs.readFile(HTML_PATH, 'utf-8');

    const html = interpolate(template, { notes: formatNotes(notes) });

    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end(html);
  });
};

const start = (notes, port) => {
  const server = createServer(notes);

  server.listen(port, () => {
    const address = `http://localhost:${port}`;
    console.log(`Server running on ${address}`);
    open(address);
  });
};

export { start };
```

Key details:

- `new URL('./template.html', import.meta.url).pathname` is a reliable, ESM-friendly way to get the absolute file path of `template.html`, relative to the current file — regardless of where the script is run from. (Note: `.pathname` can behave oddly on Windows, so it's worth knowing that's a possible gotcha.)
- The callback passed to `createServer` is marked `async` because reading the template file with `fs.readFile` is an asynchronous operation that needs to be awaited before continuing.
- `res.writeHead(statusCode, headers)` is a convenience method that lets you set the status code _and_ headers in a single call, instead of setting `res.statusCode` and calling `res.setHeader` separately.
- `start()` is the function that actually creates the server, starts it listening, logs the address to the terminal (so you're not left wondering if it's working), and opens it in the browser automatically using the small `open` npm package (`npm install open`) — purely a convenience, not a requirement.
- Only `start` is exported from this file — `createServer`, `interpolate`, and `formatNotes` stay private to the module. That's a deliberate tradeoff: keeping functions private is good encapsulation, but it also means those private functions _can't be unit tested_ from outside the file, since there's nothing to import. It's often still worth exporting internal functions specifically so they remain testable, even if you don't strictly need to use them elsewhere.

**Connecting it to the existing CLI:** the app's `web` command already accepted a `port` argument (defaulting to `5000`); the last step is simply to fetch all the notes and hand them off to `start`:

```javascript
const notes = await getAllNotes();
start(notes, argv.port);
```

Once that's wired up, running the `web` command spins up a server on the given port that displays all your notes as HTML in the browser.

## A Note on HTTP vs. Express

Everything above uses Node's raw built-in `http` module directly. In real-world projects, you'll almost always reach for a framework like **Express.js** instead, since it's far more ergonomic for building servers. But Express itself is built on top of the `http` module — so understanding how `createServer`, `req`/`res`, status codes, and headers work at this lower level gives you a solid foundation for understanding what frameworks like Express are doing for you under the hood.

---

## Summary / Key Takeaways

- **A server is just a program that listens for network requests and sends back responses** — Node's built-in `http` module (`http.createServer`) is the lowest-level way to do this, using the classic `(req, res)` callback pattern.
- **Status codes communicate outcome without reading response content**: 2xx = success, 3xx = cached/redirect, 4xx = client error, 5xx = server error. Headers (like `Content-Type`, using MIME types) tell the client what _kind_ of data is being sent.
- **Servers stay running** (`server.listen(port, callback)`) — unlike scripts you've run before, they don't exit after finishing; they wait indefinitely for the next request.
- **Interpolation is just template + data → final string**, done manually here with a regex-based `interpolate()` helper — conceptually the same thing React/Vue/Svelte do for you automatically.
- **`http` is the foundation, Express is the practical choice** — this section deliberately used the raw `http` module to build understanding, but real projects almost always use a framework like Express, which is built on top of these same primitives.