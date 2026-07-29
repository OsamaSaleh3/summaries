

## What Is a Module, Really?

Before frameworks and bundlers were common, developers would isolate their code using an **IIFE** (Immediately Invoked Function Expression):

```javascript
(function () {
  console.log("hello");
})();
```

An IIFE takes code and wraps it in its own scope — nothing outside those parentheses can see or interfere with what's inside. That's the core idea behind a module: **isolation**. A module is a self-contained block of code (think of it like a Lego block) that doesn't leak into or get disturbed by other code, but can deliberately interact with other modules when you choose to let it.

Almost every modern programming language has some concept of modules. In Node.js, there are three categories worth knowing:

1. **Internal (core) modules** — things Node ships with, like `http` or the file system module (`fs`).
2. **User-created modules** — code you write and split into reusable pieces yourself. You can also share these with the community by publishing them as packages.
3. **Third-party modules** — user-created modules made by _someone else_, which you download and use.

## ES6 Modules (`import`/`export`) vs. CommonJS (`require`)

Node.js version 18 (used in this course) supports the newer **ES6 module syntax** — the same `import`/`export` style you'd use in React or other frontend frameworks. This isn't turned on by default, though. To enable it, add this to `package.json`:

```json
{
  "type": "module"
}
```

### Creating and Using Your Own Module

Say you have a file `utils.js`:

```javascript
export function count(num) {
  return num;
}
```

Adding `export` in front of something makes it a **named export** — meaning that whoever imports it must use that exact name:

```javascript
import { count } from "./utils.js";
```

A few important details about this:

- The path must start with `./` (or `../` to go up a directory) — that's how Node knows you're pointing to a file _you_ made, not a package.
- **The `.js` extension is required** with `type: "module"` enabled. This is a relatively new requirement — for the last decade, extensions were typically optional or omitted. The reasoning: once _everything_ can be a module (JavaScript files, images, CSS files, etc.), the extension is what disambiguates which type of file you actually want.

You can also use a **default export** instead:

```javascript
export default {
  someData: 123,
};
```

With a default export, the name you use when importing it doesn't matter — you can call it anything you want:

```javascript
import data from "./utils.js";
```

A student asked whether a default export exports _everything_ in the file or just part of it. The answer: **it exports only whatever you explicitly marked as default** — whatever value sits to the right of `export default`.

### Importing Internal and Third-Party Modules

Importing a core Node module (like `fs`, for file system access) doesn't require a path — just the module's name:

```javascript
import fs from "fs";
```

On newer Node versions, you can be extra explicit that you want the _built-in_ module (as opposed to some unrelated third-party package that happens to be named the same thing) by prefixing it with `node:`:

```javascript
import fs from "node:fs";
```

This doesn't change behavior — it's purely for clarity, and it's optional.

Third-party modules (after installing them) are imported exactly the same way as core modules — just the package name, no path:

```javascript
import lodash from "lodash";
```

As your app grows, files end up importing from and exporting to each other, forming a **dependency graph**. This can even become **cyclic** (file A exports to file B, which exports to file C, which imports back from file A) — Node resolves this automatically, though it's worth being aware it can happen.

### `require` vs. `import` (CommonJS)

Since ES6 modules are relatively new to Node, a lot of existing codebases still use the older **CommonJS** system with `require` and `module.exports`. It's worth knowing the equivalents, since you'll likely encounter legacy code that uses it:

```javascript
// ES6 import
import { count } from "./utils.js";

// CommonJS equivalent
const { count } = require("./utils.js");
```

For exporting, CommonJS commonly uses `module.exports`:

```javascript
// CommonJS export
module.exports = {
  count: function () { /* ... */ },
};
```

(You may occasionally see the older `exports.count = ...` style, but it's rarely used today.)

CommonJS exists because when Node.js first came out, there was no module system in JavaScript at all — not even in the browser — so Node had to invent its own. Now that ES6 modules are standardized, Node supports both, but **`import`/`export` is the recommended, forward-looking approach.**

A student asked whether `require` or `import` is the better practice going forward. The instructor's take: use `import`, mainly for **consistency** — virtually all modern frontend tooling uses ES6 modules, so using the same style on the backend avoids the jarring experience of switching mental models (and syntax) between frontend and backend code in the same stack. It's also likely that `import`/`export` support will eventually just become Node's default behavior without needing to flip a flag in `package.json` — the current opt-in requirement is mainly there for backwards compatibility with the enormous number of existing CommonJS-based packages.

## Thinking in Modules (Code Organization Philosophy)

A key mindset shared in this section: **don't be stingy with modules.** Unlike in the browser — where every additional file might mean an extra network download, so developers minify and bundle aggressively — a Node.js backend doesn't have that constraint. There's no real performance cost to having many small files instead of one large one.

Benefits of breaking code into many small modules:

- **Easier to test** individual pieces in isolation.
- **Fewer merge conflicts** when working on a team, since changes are spread across smaller files instead of piling into one giant one.

There's no single "correct" way to organize modules — some people group files by _functionality_ (e.g., a `notes` module, a `db` module), others group by _similarity_ (e.g., a general `utilities` file for miscellaneous helper functions). Both are valid; it depends on personal or team preference.

### The `index.js` Pattern

A common organizational trick: if you have a folder full of related files, you can create an `index.js` inside that folder whose only job is to import everything from the sibling files and immediately re-export them:

```javascript
// utils/index.js
export * from "./utils.js";
export * from "./other.js";
```

Then, from outside the folder, you can import the entire folder at once:

```javascript
import * as utils from "./utils";
```

Because you imported a _folder_, Node automatically looks for an `index.js` file inside it and treats it as the entry point. Everything that file re-exports becomes available as a property on the imported object (e.g., `utils.count`, `utils.other`). This acts like a "router" for your modules — a convenient way to consolidate multiple files' exports into a single import statement elsewhere in your app.

## Useful Internal Modules

A few of Node's built-in modules worth knowing:

- **`fs`** (file system) — lets you read files, write files, create directories, and generally do almost anything you could do with files manually, but programmatically. This is powerful enough that you could build your own simple database with it (which is exactly what happens later in the course).
- **`http`** — Node's built-in module for basic networking, used to build servers. It's a bit low-level; most real-world apps use a framework built on top of it rather than using `http` directly.
- **`path`** — used for working with file paths (mentioned briefly).

## Understanding `npm`

**npm** stands for **Node Package Manager**. Its job is simple: manage the packages (third-party modules) your project depends on.

To install a package:

```bash
npm install exif-parser
# or the shorthand:
npm i exif-parser
```

A good way to find packages for almost anything you want to do: search "npm" plus whatever functionality you need — there's almost always an existing package for it.

Installing a package creates two important things:

- A **`node_modules`** folder — where all downloaded packages actually live on disk.
- A **`package-lock.json`** file — this locks in the _exact_ versions of everything installed (including nested dependencies), ensuring that everyone on a team, and every deployment environment, ends up with identical package versions.

Your `package.json` also gets a `dependencies` object listing what you installed, but the version number typically has a **caret (`^`)** in front of it, meaning "anything within this compatible range" — not an exact pin. That's why `package-lock.json` exists: to guarantee an exact, reproducible version across machines.

**You should never commit `node_modules` to Git.** Reasons given:

- It would bloat every pull request with changes to code you didn't even write.
- It's completely unnecessary — `package.json` and `package-lock.json` already contain everything needed to recreate `node_modules` from scratch with a single command:

```bash
npm install
```

Running `npm install` with no arguments reads those two files and downloads everything listed, recreating an identical `node_modules` folder — this is exactly what happens when a new team member pulls down the project for the first time.

To remove a package:

```bash
npm uninstall exif-parser
```

This removes it from `package.json`, `package-lock.json`, and `node_modules` all at once.

A student asked about the security risk of installing random npm packages. The honest answer: **yes, this is a real risk**, and it cuts both ways — sometimes it's a malicious package, but far more often it's simply a package that's broken or abandoned (a maintainer moves on, a dependency shifts underneath it, and it stops working). Larger companies often maintain a whitelist of approved packages, sometimes even involving legal review for licensing concerns. A practical way to vet a package before installing it: check its GitHub repository's recent activity and open issues — if something is seriously broken or unsafe, there's a good chance someone has already flagged it there.

## Bringing In `yargs` for Command Parsing

To avoid manually parsing `process.argv` with a pile of `if` statements and regular expressions, the section introduces a popular third-party package: **yargs**. Its whole purpose is to parse command-line arguments and turn them into a clean, well-structured interface — including auto-generating a `--help` menu, something virtually every real CLI provides.

After installing it (`npm install yargs`), and organizing the CLI logic into its own file (e.g. `source/command.js`, imported from a now near-empty `index.js`), the basic setup looks like this:

```javascript
import yargs from "yargs";
import { hideBin } from "yargs/helpers";

yargs(hideBin(process.argv))
  .command(/* ... */)
  .demandCommand(1)
  .parse();
```

- **`hideBin(process.argv)`** simply strips off the first two entries of `process.argv` (the Node path and file path), leaving just the actual arguments the user typed.
- **`.demandCommand(1)`** requires at least one command to be provided — running the CLI with no command at all will produce an error instead of doing nothing.
- **`.parse()`** tells yargs to actually process the arguments and run the matching command.

A useful architectural note here: because `index.js` just does one line — `import "./source/command.js";` — that single import statement executes the whole file. This means the hashbang comment (`#!/usr/bin/env node`) only needs to live at the very top of the **entry file**, not every file in the project, since all imported files run within that same process.

### Defining Commands

A command is registered with four arguments: the command name (and any expected arguments), a description, an optional builder function (for configuring arguments), and a handler function (what actually runs):

```javascript
yargs(hideBin(process.argv))
  .command(
    "new <note>",
    "create a new note",
    (yargs) => {
      return yargs.positional("note", {
        type: "string",
        describe: "the content of the note to create",
      });
    },
    (argv) => {
      console.log(argv.note);
    }
  )
  .parse();
```

- **Angle brackets** (`<note>`) mean the argument is **required**.
- **Square brackets** (`[port]`) mean the argument is **optional** — typically paired with a default value if the user doesn't supply one.

You can also add **options** (flags) to a command:

```javascript
.option("tags", {
  alias: "t",
  type: "string",
  describe: "comma separated tags for the note",
})
```

This lets a user pass something like `--tags work,serious` (or the shorthand `-t work,serious`, thanks to the alias), and the parsed value becomes available on `argv.tags`.

Running the CLI with `--help` (something every yargs-based CLI gets automatically) lists out all registered commands, arguments, and options — including any new ones you add later, with zero extra maintenance work.

---

## Key Takeaways

- **A module is a way to isolate code** — conceptually descended from the old IIFE pattern — and Node has three kinds: internal (built-in), user-created, and third-party.
- **ES6 `import`/`export`** is the modern, recommended syntax (enabled via `"type": "module"` in `package.json`), while **CommonJS (`require`/`module.exports`)** is the older system you'll still encounter in legacy Node code — know both, prefer `import`/`export` going forward.
- **Don't be stingy with modules** — small, focused files are easier to test and reduce merge conflicts, and there's no real performance downside on the backend. The `index.js` pattern lets you consolidate a folder's exports into a single import.
- **`npm`** manages dependencies: `npm install <pkg>` adds a package (creating `node_modules` and `package-lock.json` for reproducible installs), `npm uninstall <pkg>` removes it, and `node_modules` should never be committed to Git.
- **`yargs`** turns raw `process.argv` parsing into a clean, declarative command structure — complete with required/optional arguments, aliased flags, and an automatically generated `--help` menu.