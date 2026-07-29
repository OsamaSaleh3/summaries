

## What Is a CLI?

A CLI (**Command Line Interface**) is simply a program that runs in your terminal. You've been using CLIs the whole time without necessarily thinking of them that way — commands like `ls`, `cd`, `mkdir`, and `rm` are all CLIs, either built into your operating system or installed separately.

A helpful mental model: a CLI is like chatting with your computer, except instead of natural language, you communicate through specific commands that trigger code to run.

An important point: **a CLI can be written in literally any programming language** that runs on your machine — not just Node.js. You could write one in Bash, Python, C#, whatever. And when you _run_ a CLI, it doesn't matter what language built it, as long as your operating system has the right runtime available to execute it. Many CLIs already on your computer probably weren't built with Node or JavaScript at all.

## The `process` Global and Its Environment

Node.js runs inside your operating system's environment (rather than a browser environment), and one of the most powerful things this gives you access to is a global object called **`process`**. It exposes a ton of information about the environment your code is currently running in — details about the machine, other running programs, and so on. This is what lets a Node app behave dynamically depending on which computer it ends up deployed to, since Node apps are typically meant to run on servers/computers other than the one they were written on.

### `process.argv`

One of the most useful properties on `process` is **`argv`** — it captures everything typed after the `node` command when you run a file.

```javascript
console.log(process.argv);
```

Running this file like so:

```bash
node index.js thing thing2 42
```

produces an array where:

- The **first item** is always the path to the Node executable being used.
- The **second item** is always the path to the file that was executed.
- **Everything after that** is, in order, whatever arguments you passed.

This is essentially how you pass arguments into a terminal program, and it's the foundation for making a CLI dynamic — reacting differently depending on what arguments a user supplies.

### `process.env`

`process.env` gives you access to **environment variables** — the standard way of storing configuration values (like API keys or secrets) that shouldn't be hard-coded into your source code or checked into GitHub. You typically set these on whatever platform you're deploying to (AWS, Vercel, etc.), and then read them in your code like:

```javascript
process.env.SOME_VARIABLE_NAME
```

Every operating-system-level language has some equivalent of this, because it's essentially impossible to safely deploy an app to a remote server without a way to inject secrets outside of your codebase.

One commonly used environment variable is **`NODE_ENV`**, which the community uses to flag what "mode" the app is running in — typically `development`, `production`, or `testing`. Your code can check this value and change behavior accordingly: for example, turning on verbose logging in development but disabling it in production, or bypassing authentication locally while enforcing it in production. Frameworks like React also check `NODE_ENV` to decide whether to show extra warnings (development) or optimize for performance and suppress warnings (production).

A student asked whether Node has something like `process.argc` (a concept from other languages, used for _counting_ arguments). The answer: **no, there's no `argc` in Node** — you'd just use `process.argv.length` if you needed a count.

Another question that came up: what's the best practice for sharing `.env` files across a team? There's no single perfect answer — approaches vary:

- Some teams use dedicated secret-management tools/apps that securely distribute environment variables via a script and a single shared credential.
- Some use encrypted password managers (like a shared vault).
- In many cases, each team member creates their _own_ set of credentials (e.g., their own database user), so there's nothing sensitive to share at all.
- GitHub now proactively scans for and warns about exposed secrets in pull requests, which helps — but the problem isn't fully solved. Leaked environment variables have been the root cause of some of the biggest security breaches in the industry.

## Setting Up a Node Project

To turn a plain folder into a proper Node.js project, you use **`npm`** (which comes bundled with Node — you don't need to install it separately):

```bash
npm init --yes
```

`npm init` asks a series of setup questions (project name, version, etc.) and then generates a **`package.json`** file. Passing `--yes` skips all the prompts and accepts the defaults. This file is Node's equivalent of the configuration files found in other languages — it's where dependencies, scripts, and other project metadata live. For a simple app (rather than a published package), most of the auto-generated fields don't really matter.

## Turning a Script Into a Real CLI Command

Having `index.js` log `"hello world"` isn't a CLI yet — it's just a script you run with `node index.js`. To make it a proper command you can type directly into your terminal (like `note`), you need two things:

**1. A `bin` field in `package.json`:**

```json
{
  "bin": {
    "note": "./index.js"
  }
}
```

This tells Node: "create an entry named `note` in the system's CLI directory (the `bin` folder), and point it at this file." You want to pick a globally unique name — naming your CLI something like `git` (which already exists as a real CLI) would cause conflicts.

**2. Linking it locally with `npm link`:**

```bash
npm link
```

This creates a **symlink** (a virtual reference, not a full copy) between your project files and your system's CLI folder. The advantage of a symlink over a real install: every time you edit your code and save, the command picks up the change immediately — you don't have to reinstall anything.

At this point, running the command (e.g., typing `note`) will likely produce an error like:

```
syntax error near unexpected
```

This isn't a JavaScript syntax error — the file itself runs fine with `node index.js`. The real problem is that your operating system doesn't know _what runtime_ should execute this file. It sees a file and has no way of knowing it's JavaScript rather than Python, C#, or anything else.

The fix is to add a **hashbang** (also called a shebang) as the very first line of the file:

```javascript
#!/usr/bin/env node
```

This line isn't a normal JavaScript comment (`//` or `/* */`) — the JavaScript interpreter never even sees it. It's a special instruction meant for the operating system, telling it exactly what runtime to use to execute the file. It has to be the literal first thing in the file — not even a blank line can come before it.

If, after all this, typing your command still says **"command not found"**, that typically means the symlink didn't get created. Checking `which <your-command-name>` will confirm whether it's linked; if it says "not found," you need to run `npm link` again.

## Capturing Input to Build a Simple Note

With the CLI wired up, you can start capturing user input through `process.argv`. For example, typing:

```bash
note "this is my new note"
```

The note content will land at `process.argv[2]` (remember, indexes 0 and 1 are always the Node path and file path):

```javascript
const note = process.argv[2];

const newNote = {
  content: note,
  id: Date.now(),
};

console.log(newNote);
```

Running this produces an object with the note content and a generated ID.

One detail worth remembering: the note content has to be wrapped in quotes in the terminal. Without quotes, each space-separated word would be treated as a **separate** argument (so `note this is my new note` would produce five separate arguments instead of one string). Wrapping it in quotes tells the terminal, "treat this whole block as a single argument."

At this stage, the CLI is still very limited — it can't add tags, search notes, or retrieve anything, and critically, **nothing is being saved anywhere**. Once the program finishes executing, the data in memory is gone. The next section builds toward persisting this data and relying on Node's modules (both built-in and third-party) instead of parsing everything by hand.

---

## Key Takeaways

- **A CLI is just a program that runs in the terminal** and can be written in any language — Node.js is only one option among many.
- **`process.argv`** gives you the raw list of terminal arguments (with the Node path and file path always occupying the first two slots), and **`process.env`** gives you access to environment variables like `NODE_ENV`, which is commonly used to toggle behavior between development, testing, and production.
- **`package.json`** is created via `npm init` and is central to configuring a Node project; the **`bin`** field plus **`npm link`** is what turns a script into an actual terminal command.
- **A hashbang (`#!/usr/bin/env node`)** must be the very first line of your entry file so the operating system knows which runtime should execute it.
- Quoting matters: unquoted, space-separated terminal input becomes multiple arguments, not one string — wrap multi-word input in quotes to keep it as a single argument.