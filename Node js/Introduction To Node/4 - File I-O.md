
## Asynchronous JavaScript in Node (The Buildup)

Before touching the file system, this section spends time on _why_ asynchronous code exists and how it evolved — because you'll do dramatically more async programming in Node than you ever did with client-side JavaScript.

**Async code just means: things don't necessarily run in the exact order they were written.** Important nuance the instructor was careful to clarify: this is _not_ the same as things running "at the same time." By default, Node isn't doing multiple things simultaneously (that would require opting into workers/threads) — it's scheduling things to happen _later_. From the outside, it can feel like things are happening in parallel, but under the hood it's really about deferring and registering work, not true simultaneous execution.

A quick "trick question" illustrated an important distinction: having a callback doesn't automatically mean something is asynchronous.

```javascript
new Array(20)
  .fill(0)
  .map((_, i) => console.log(i));
```

Even though `.map()` takes a callback, this code is **100% synchronous** — nothing about it is async. The instructor's rule of thumb: something is asynchronous almost exclusively because of one of three things:

1. You're doing something over the **network**.
2. You're using a **timing function** (`setTimeout`, `setInterval`).
3. You're interacting with **storage** — the file system or a database.

99% of the async code you'll write in Node falls into one of those three buckets.

### From Callbacks → Promises → Async/Await

**Callbacks** are the original way to handle async code — you pass a function to be called once some async work finishes. The problem: nesting many async steps that depend on each other leads to deeply nested, hard-to-read code often called **"callback hell"** (or the "pyramid of doom") — a callback inside a callback inside a callback, endlessly indenting to the right.

**Promises** were introduced to flatten this out:

```javascript
someAsyncThing().then(() => {
  console.log("done");
});
```

You still get a callback with `.then()`, but critically, you're only ever nested **one level deep** — you can keep chaining `.then()` calls indefinitely instead of nesting further and further inward.

You can build your own Promise around any callback-based function:

```javascript
function wait(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, ms);
  });
}

wait(3000).then(() => {
  console.log("3 seconds passed");
});
```

Node also ships a built-in **`promisify`** utility that can automatically convert an older callback-style function into a Promise-based one, which is handy when working with legacy code from before Promises were standard.

**Async/await** is the final evolution — it lets asynchronous, Promise-returning code _read_ like ordinary synchronous code:

```javascript
async function run() {
  await wait(3000);
  console.log("3 seconds passed");
}
```

The rule: put `await` in front of anything that returns a Promise, and the next line simply won't execute until that Promise resolves. This is functionally equivalent to chaining `.then()`, just far more readable.

One especially useful Node detail: newer versions support **top-level await** — meaning you don't have to wrap your code in an `async function` just to use `await`; you can use it directly at the root of a file. (This isn't possible in browser JavaScript by default in the same way.)

A student asked a great question: _why bother making async code "synchronous" with an extra step (async/await) at all?_ The answer given: in a trivial example it might not seem worth it, but in real applications — say, a route handler that checks a database, creates a customer record in a payment system, and reports to analytics — you end up with many async operations that depend on each other. The more deeply those get nested with raw callbacks or chained `.then()`s, the harder the code becomes to reason about, which commonly leads to bugs like **race conditions**. A common (bad) workaround people reach for is throwing in an arbitrary `setTimeout` to "fix" timing bugs — which doesn't actually solve anything, since you can never be sure how long is "long enough." Async/await sidesteps all of this: you know for certain the next line won't run until the previous one finishes, making the code far easier to reason about — even though behind the scenes, async/await is really just syntax built on top of JavaScript **generators**.

## The `fs` Module

**`fs`** stands for **file system** — a core Node module that gives you a full API for interacting with files: creating directories (`mkdir`), reading directory contents (`readdir`), getting file info (`stat`), deleting files (`unlink`), renaming, and — the two used most — **reading** and **writing** files.

### Reading a File

```javascript
import fs from "node:fs/promises";

const pjsonPath = new URL("./package.json", import.meta.url);

const readPjson = async () => {
  const contents = await fs.readFile(pjsonPath);
  console.log(contents);
};
```

A few important details buried in this example:

- **`import.meta.url`** (used to build a path) exists because, once you're using `"type": "module"`, you lose access to the old Node global **`__dirname`**, which used to give you the current working directory automatically. `import.meta.url` combined with `new URL(...)` is the modern replacement for constructing an absolute path.
- **You can't just pass a relative path** like `"./package.json"` directly into `readFile` — you need something closer to an absolute path, which is exactly what this `new URL(...)` construction produces.
- There are multiple versions of `readFile`:
    - **`fs.readFileSync`** — the **synchronous**, blocking version. This should generally be avoided in real applications: if a server were handling 100,000 requests per second and one route synchronously reads a file, _every other request has to wait_ for that read to finish.
    - **`fs.readFile`** (callback-based, from `"fs"`) — asynchronous, but requires a callback, which isn't the preferred modern style.
    - **`fs.promises.readFile`** (or importing from `"fs/promises"`) — the Promise-based version, which pairs naturally with `async`/`await`. This is the recommended approach.

### Writing a File

```javascript
import fs from "node:fs/promises";

const newFilePath = new URL("./demo.js", import.meta.url);

async function writeDemo() {
  await fs.writeFile(newFilePath, "console.log('yoooo!')");
}

writeDemo();
```

This writes an actual JavaScript file to disk, which could then be executed separately (e.g., chaining commands with `&&` in the terminal to write the file and then immediately run it). This is the same basic mechanism used by tools like Create React App to scaffold new project files for you.

A student asked how to debug deeply nested callback issues — especially when the nesting spans multiple modules rather than staying in one file. The instructor's approach, in short: rather than relying heavily on a debugger, work from the **innermost** call outward, logging your _expectation_ at each step and comparing it to what actually happens. If there are many layers of abstraction making it hard to even locate where something is happening, that in itself may be a sign you need a framework (or better structure) to manage those abstractions for you. Most bugs come down to the code doing exactly what you told it to do — just not what you _meant_ to tell it.

Another question: does `readFile` open a file handle and then close it afterward? **Yes** — it's not a persistent stream; the file is opened, fully read, and closed. This matters because opening and closing file handles is relatively expensive, which is part of why file operations are asynchronous by default.

## Using the File System as a Simple Database

Rather than connecting to a real database, this section uses a plain JSON file as a lightweight, file-based "database" for storing notes.

**1. Create `db.json`** at the project root:

```json
{
  "notes": []
}
```

**2. Create `db.js`** — a set of reusable utility functions that abstract away all direct interaction with that file. The instructor compares this layer to an **ORM** (a way of interacting with a database without writing raw queries by hand) — except here, the "database" is just a file.

```javascript
import fs from "node:fs/promises";

const DB_PATH = new URL("./db.json", import.meta.url);

export const getDB = async () => {
  const db = await fs.readFile(DB_PATH, "utf-8");
  return JSON.parse(db);
};

export const saveDB = async (db) => {
  await fs.writeFile(DB_PATH, JSON.stringify(db, null, 2));
  return db;
};

export const insertDB = async (note) => {
  const db = await getDB();
  db.notes.push(note);
  await saveDB(db);
  return note;
};
```

A few notes on the code above:

- **`"utf-8"`** is the text **encoding** used when reading the file — think of encoding as different "languages" computers use to represent characters (an emoji, for example, uses a different underlying representation than a plain alphabet character). `utf-8` is essentially the standard encoding for readable human text.
- **`JSON.parse`** turns a JSON string into a real JavaScript object; **`JSON.stringify`** does the reverse. The extra arguments in `JSON.stringify(db, null, 2)` tell it to format the output with 2-space indentation, so the saved file stays human-readable instead of being crammed onto a single line.
- Why not use **`fs.appendFile`**? Because append just blindly adds raw text to the end of a file — it has no concept of JSON structure, so it would corrupt the array format. Instead, `insertDB` reads the whole file, parses it into a real array, pushes the new item, and writes the whole thing back.

Because this project is a single-user, local CLI tool, a student's question about handling concurrent reads/writes (e.g., one process reading while another is inserting) was explicitly out of scope — that's a real concern in multi-user, networked apps, but not something this simple, single-person tool needs to account for.

## Building a Notes-Specific Abstraction (CRUD)

**CRUD** stands for **Create, Read, Update, Delete** — the instructor's point being that the overwhelming majority of applications you'll ever build boil down to some version of a CRUD app.

Rather than using the generic `db.js` functions directly inside the CLI commands, the section adds one more layer of abstraction specific to _notes_ (`notes.js`), keeping the database functions themselves generic and reusable (in case, say, a `users` collection were added later).

```javascript
import { getDB, saveDB, insertDB } from "./db.js";

export const newNote = async (note, tags) => {
  const newNote = {
    content: note,
    id: Date.now(),
    tags,
  };

  await insertDB(newNote);
  return newNote;
};

export const getAllNotes = async () => {
  const { notes } = await getDB();
  return notes;
};

export const findNotes = async (filter) => {
  const { notes } = await getDB();
  return notes.filter((note) =>
    note.content.toLowerCase().includes(filter.toLowerCase())
  );
};

export const removeNote = async (id) => {
  const { notes } = await getDB();
  const match = notes.find((note) => note.id === id);

  if (match) {
    const newNotes = notes.filter((note) => note.id !== id);
    await saveDB({ notes: newNotes });
    return id;
  }
};

export const removeAllNotes = () => saveDB({ notes: [] });
```

Walking through the reasoning behind each function:

- **`newNote`** takes the note content and tags, builds a new object with a generated `id` (using `Date.now()` as a simple unique-enough identifier), and uses `insertDB` — deliberately _not_ `saveDB` — because `insertDB` only adds one note without wiping out the rest of the database, whereas `saveDB` overwrites the entire file.
- **`getAllNotes`** just reads the database and returns the `notes` array — almost "free," since `getDB` already does the heavy lifting.
- **`findNotes`** performs a very simple substring search using `.includes()` (described candidly as "the poor person's searching"). Both the note content and the search filter are lowercased first, so matching isn't case-sensitive.
- **`removeNote`** first checks whether a note with the given `id` actually exists. If it does, a **new array is created via `.filter()`** that excludes the matching note, and that new array is saved back — the original note isn't literally deleted in place; a fresh array without it is saved instead.
- **`removeAllNotes`** is a one-liner that just resets `notes` back to an empty array. Because there's no code that needs to run _after_ this call finishes, it doesn't need `await` at all — you can simply `return` the Promise, and whoever calls this function can `await` it whenever they use it.

A student asked why the instructor returns _new_ arrays instead of mutating the existing ones in place. The reasoning: it's a long-standing habit to avoid **mutation** because it can introduce subtle side effects, and creating new objects/arrays tends to be easier to reason about, especially as logic grows more complex. That said, immutability isn't a universal rule — in tightly memory-constrained environments (the instructor gave the example of building for a TV, where creating lots of new objects is comparatively expensive), mutating in place can actually be the more sensible choice. It's a tradeoff, not a hard law.

### A Detour on Destructuring

Because destructuring shows up throughout this code (`const { notes } = await getDB();`), it's worth understanding clearly. Instead of writing this:

```javascript
const jumping = data.jumping;
const shooting = data.shooting;
```

you can pull multiple properties off an object in a single line:

```javascript
const { jumping, shooting } = data;
```

The variable names you use **must match the property names** on the object. This works with arrays too:

```javascript
const [first] = nums; // grabs the first item
```

Destructuring also works directly inside function parameters — letting you pluck out just the properties you care about, and optionally gather everything else using the **rest/spread syntax** (`...rest`):

```javascript
function action({ this: thisThing, that, other, ...rest }) {
  // ...
}
```

## Wiring the Commands Up

The final step connects these `notes.js` functions to the actual yargs commands defined earlier, replacing the placeholder logging with real logic. For example, the `new` command becomes:

```javascript
const tags = argv.tags ? argv.tags.split(",") : [];
const note = await newNote(argv.note, tags);
console.log("new note", note);
```

Since **tags arrive as a single comma-separated string** from the command line (e.g. `--tags work,serious`), they need to be split into an actual array before being passed along — and if no tags were provided at all, an empty array is used as the fallback.

A student asked what happens if no tag is passed — does it get stored as `undefined`? The answer: it depends on where you handle the default. In this case, it defaults to an empty array rather than `undefined`. It's worth noting that even if a value _were_ `undefined`, `JSON.stringify` strips `undefined` properties out entirely when saving — whereas `null` would be preserved in the saved file.

With all the commands wired to their respective `notes.js` functions (`new`, `all`, `find`, `remove`, and `clean` for wiping everything), the CLI becomes a genuinely functional, persistent note-taking tool — notes survive between runs because they're saved to `db.json` on disk, rather than disappearing the moment the process exits.

---

## Key Takeaways

- **Async code in Node almost always comes down to three things**: network requests, timers, or file/storage access — and the progression from **callbacks → Promises → async/await** exists purely to make that async code easier to read and reason about, not to change what's actually happening under the hood.
- **The `fs` module** (especially the Promise-based version, `fs/promises`) is the tool for reading and writing files; prefer the async Promise-based methods over `readFileSync`/callback-based versions, since synchronous file reads block everything else.
- **A JSON file can act as a simple "database"** by combining `readFile` + `JSON.parse` (get the data), and `JSON.stringify` + `writeFile` (save the data) — with `insertDB`-style functions used to add a single item without overwriting the whole dataset.
- **Layering abstractions matters**: generic database functions (`getDB`/`saveDB`/`insertDB`) stay reusable for any kind of data, while a separate notes-specific layer (`newNote`, `findNotes`, `removeNote`, etc.) implements the actual CRUD logic your app needs.
- **Destructuring** (`const { notes } = await getDB()`) is a shorthand for pulling multiple properties off an object (or items off an array) in one line, and pairs naturally with the async patterns used throughout this section.