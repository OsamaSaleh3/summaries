

## What Node.js Actually Is

Before Node.js, JavaScript could only run inside a browser. Languages like Python, Ruby, and PHP could run directly on your computer through a terminal, but JavaScript couldn't — it was seen as a "browser-only" toy language. Node.js changed that by taking Google's V8 engine (the thing that runs JavaScript inside Chrome) and embedding it into a standalone program that runs on your computer, outside the browser.

The key idea to internalize: **Node.js is not a language. It's a runtime/environment.** JavaScript is the language. This is actually unusual — most languages (Python, Ruby) are their runtime. JavaScript is one language that runs in two different runtimes: the browser and Node.js. Same language, same syntax, same functions — but each runtime exposes different capabilities (globals, APIs) depending on where it's running.

This is why **if you already know JavaScript, you already know about 90% of Node.js.** The syntax doesn't change. What changes is _what you have access to_. The browser gives you things like the DOM and `alert()`. Node gives you things like file system access and process information. Neither environment gets the other's toys.

### A Bit of History

Node.js has been around since 2009. There was a period (around version 0.12) where development stagnated, so the community forked it into a project called **IO.js** out of frustration — which pressured the original Node.js team to get back on track with regular updates. Eventually the two merged back together, and Node has kept up a steady release cadence ever since, staying in sync with ECMAScript (the JavaScript spec) and features happening in the browser.

Today, Node powers a huge chunk of the JavaScript ecosystem — not just servers, but build tools too. If you've used React, Vite, or any frontend framework, Node was running behind the scenes converting your JSX/code into something the browser understands.

## Non-Blocking I/O and the Event Loop

This is one of the most important concepts in the whole section.

**I/O** just means input/output — things going into or out of your program (reading a file, making a network request, etc.).

In most traditional languages (Ruby, Python, PHP), if you run a piece of code, nothing else can happen until it's done — unless you spin up an entirely new thread, which costs system resources. JavaScript (and Node specifically) is different: it can handle things concurrently on a **single thread**, using something called the **event loop**.

Think of the event loop as a registry for pending work. When you kick off a task (like reading a file or waiting on a timer), Node registers that work and moves on immediately to the next line of code. Once that background work finishes, the event loop calls back into your code to let you know it's done.

This is what makes Node good at handling lots of concurrent requests — a server built in Node can often juggle more simultaneous connections out of the box than other frameworks, without extra optimization work. But this is _not_ the same as saying Node is "faster." Node is intentionally single-threaded, so it's a poor fit for CPU-heavy work like machine learning — those tasks benefit from using every core available, and Node doesn't do that by default (which is part of why languages like Python are more common there).

An important clarification the instructor added: **the event loop in Node and the event loop in browser JavaScript work the same way from a developer's perspective.** The underlying implementation differs (they're managing different system resources), but how you experience and reason about it as a programmer is identical.

### Blocking vs. Non-Blocking Code

Blocking (synchronous) code runs top to bottom, one line finishing completely before the next starts:

```javascript
function getUser(id) {
  // a normal, instant, synchronous lookup
  return { id, name: "Scott" };
}
```

Non-blocking (asynchronous) code lets you schedule work to happen later, without freezing everything else:

```javascript
function getUser(id) {
  setTimeout(() => {
    // this callback runs ~1000ms later
    return { id, name: "Scott" };
  }, 1000);
}
```

If you have code _after_ this function call, it will execute immediately — it won't wait for the 1-second timer. That's the essence of non-blocking behavior: work gets registered, the rest of your program keeps going, and the registered work is handled whenever it's ready.

## Browser Globals vs. Node Globals

In the browser, everything implicitly lives on a global object called `window`. When you type `alert("hi")`, you're really calling `window.alert("hi")` — you just never had to write `window.` because it's assumed.

Node has no `window` — trying to reference it throws `window is not defined`. Instead, Node has its own global object, literally called `global`. You'll almost never use `global` directly (just like you rarely type `window.` in the browser), but it exists, and it contains Node equivalents of familiar things like `setTimeout`, `setInterval`, `clearInterval`, and even a built-in `fetch`.

Some other key differences:

- **No DOM.** There's no `document`, no HTML elements to reference, because there's no visual page — Node's output is text in a terminal, not a rendered browser window. Trying to use DOM methods in Node will simply error out.
- **`alert()` doesn't exist in Node**, because it's meant to produce a visual dialog box — something that only makes sense in a browser.
- **`console` works the same way in both**, but it means different things: in the browser it prints to the browser's dev tools console; in Node it prints straight to your terminal. This is Node/JavaScript's equivalent of `print` in other operating-system-level languages.
- **Node can create servers.** A server is basically a remote computer that listens for requests and sends back data, files, or responses. Browser JavaScript is typically the _client_ — the thing sending requests to a server, not the one running as a server itself.
- **Modules work slightly differently.** Both browser and Node support the `import`/`export` module syntax, but in the browser, you declare a module using `<script type="module">`; in Node there's no HTML, so you just use `import`/`export` directly in your JavaScript files. (Modules are covered in more depth in a later section.)

## Running JavaScript With Node

To execute a plain JavaScript file with Node, you write your code in a file (e.g., `index.js`) and run it from the terminal:

```bash
node index.js
```

This is fundamentally different from the browser workflow, where you'd need a `<script>` tag, an HTML page, and a browser to open the console in. With Node, any arbitrary JavaScript file can just be executed directly from your terminal.

## The Node REPL

**REPL** stands for **Read, Evaluate, Print, Loop**. It's a way to type JavaScript directly into your terminal, one line at a time, and see immediate results — essentially the terminal equivalent of opening your browser's console.

You start it by typing `node` with no file argument. From there you can write and evaluate arbitrary JavaScript, with autocomplete included. To exit, you can press `Ctrl+C` twice, use `Ctrl+D`, or type `.exit`.

The REPL is not a place to build applications — nothing you write there is saved anywhere, and once the process ends, the memory is wiped. It's mainly useful for quickly testing small snippets or doing quick calculations (the instructor mentioned occasionally using it as a calculator, since it can be faster than opening a calculator app).

---

## Key Takeaways

- **Node.js is a runtime, not a language** — JavaScript is the language; Node and the browser are two different environments that run it, each with different available globals and capabilities.
- **Non-blocking I/O + the event loop** let Node handle many operations concurrently on a single thread, which makes it great for I/O-heavy apps (servers) but not ideal for CPU-intensive work like machine learning.
- **Blocking code runs strictly in order; non-blocking code schedules work to run later** (e.g., via `setTimeout`) while the rest of the program keeps executing.
- **Browser vs. Node globals differ**: no `window`, no DOM, no `alert()` in Node — instead there's `global`, file system access, and the ability to build servers.
- **The REPL** is a quick scratchpad for testing JavaScript line-by-line in the terminal, not a tool for building real applications.