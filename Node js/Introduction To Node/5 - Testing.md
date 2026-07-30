
## Types of Tests (the big picture)

Before writing any code, it helps to know the different "flavors" of testing you'll hear about, because they answer different questions:

- **Unit testing** — You take one small unit of code (a function, a module, anything small) and test it in isolation. "If I give it this input, do I get this output?" It doesn't care about the rest of the app.
- **Integration testing** — Instead of testing one function, you test a _flow_ made of several functions calling each other (e.g. a signup flow). Each function might pass its own unit test individually, but that doesn't guarantee the whole flow works together correctly — a function could be getting the wrong inputs even though it "adds numbers correctly" internally.
- **End-to-end (E2E) testing** — This goes even further than integration testing: it simulates a real user, starting from a click on the front end, all the way through the server, the handler, the database, and back with a response. Because you're simulating real user actions (clicks, scrolling, typing), E2E testing usually needs a **browser**. You'll often hear the term **headless browser** here — it's a normal browser, just without a visible UI. The benefit is that it's faster, more memory-efficient, and can run anywhere without a display (like your terminal or a CI server), while still executing real browser code so you can test it.
- **API testing** — Similar to integration testing, but focused specifically on whether an API route behaves correctly end-to-end: correct status codes, correct headers, correct error handling, etc. Integration testing usually only cares about the logic that runs when the API is hit; API testing cares about the _entire contract_ of that one API call.
- There are other, more specialized types too (regression testing, snapshot testing, etc.), but in practice, most teams only rigorously do all of these if their organization has a strong testing culture — it's a lot of work, and not everyone loves writing tests even though everyone wants to _have_ them.

## Setting Up Jest

For this course, unit testing makes the most sense, so the tool used is **Jest** — a testing framework originally built by Facebook, which took ideas from earlier frameworks like Mocha and Jasmine and became a widely-adopted standard. It works for backend and frontend code alike.

**Setup steps:**

1. Create a `tests` folder at the project root.
2. Name test files with a `.test.js` suffix (e.g. `notes.test.js`) — Jest automatically looks for files matching this pattern using Node's built-in `fs` module to scan your project.
3. Install Jest as a **devDependency**:
    
    ```
    npm install --save-dev jest
    ```
    
    A devDependency is a dependency you only need _while developing_ (e.g. for testing), not one needed to actually run the app in production. Keeping them separate matters because production environments — or people installing your package from npm — shouldn't have to download tools they'll never use.
4. Replace the placeholder `test` script in `package.json` with Jest, then run tests with:
    
    ```
    npm test
    ```
    
    (`test` is a special script name — unlike other scripts, you don't need to type `npm run test`, just `npm test`.)

### Anatomy of a basic test

Jest wraps your test code in its own environment and gives you global helper functions like `test` and `expect`:

```javascript
function add(a, b) {
  return a + b;
}

test('add takes two numbers and returns a sum', () => {
  const result = add(1, 2);
  expect(result).toBe(3);
});
```

- `test(name, callback)` describes what you're testing.
- `expect(value)` is Jest's built-in assertion library — you chain "matchers" onto it, like `.toBe(3)`, to check whether the value is what you expected.
- If the assertion fails, Jest tells you clearly what was expected vs. what was actually received, which makes debugging fast.

This connects to the idea of **TDD (test-driven development)** and the "red-green-refactor" cycle: write your tests first (they'll fail — red), then write just enough code to make them pass (green), then clean up (refactor). In practice, most people don't rigorously follow this, but some companies do care a lot about it — and about **test coverage**, a percentage representing how much of your code your tests actually exercise. Tools can enforce a minimum coverage threshold (e.g. 80%) and will fail the build if you fall short, even highlighting exactly which lines/branches (like an untested `else`) weren't covered.

## Mocking (and why ESM makes it trickier)

When you unit test a function, you often don't want to test _everything it touches_ — just the function itself. For example, if a function saves a note by calling a database helper, you don't want your test to actually write to and read from a real file or database every time you run it. That's slow, and it's also not your job to test that the database itself works correctly — someone else already tested that.

The solution is **mocking**: replacing a real function with a fake stand-in ("stub") that does nothing (or returns fake data) instead of the real thing. A mock is also a **spy** — it remembers everything that happened to it: how many times it was called, with what arguments, etc., so you can write assertions like "I expect this function to have been called 3 times with these arguments."

Because this project uses ES Modules (`"type": "module"` in `package.json`) — which is still relatively new in Node — mocking requires some extra ceremony compared to CommonJS:

```javascript
import { jest } from '@jest/globals';

// Mock everything exported from the db module
jest.unstable_mockModule('../db.js', () => ({
  insertDB: jest.fn(),
  getAllNotes: jest.fn(),
  removeNote: jest.fn(),
}));

// Because we mocked the module, we now must import it *dynamically*,
// using `await import()`, instead of a normal static `import` statement.
const { insertDB } = await import('../db.js');
const { newNote } = await import('../notes.js');
```

The reason for the dynamic `await import()` is that the mock has to be registered _before_ the real module (and anything that depends on it) gets imported. With a normal static `import`, everything gets loaded immediately and you'd miss your chance to swap in the mock. (With CommonJS's `require`, this whole problem doesn't exist — it's a one-liner. ESM mocking is a genuinely newer, less common pattern.)

Since tests shouldn't depend on each other's leftover state, Jest gives you a `beforeEach` hook that runs before every individual test — commonly used to reset mocks so each test starts fresh:

```javascript
beforeEach(() => {
  jest.clearAllMocks();
});
```

If you don't do this, tests can accidentally rely on the order they run in, which breaks the moment someone reorders or adds a new test.

### Writing an actual mocked test

```javascript
test('newNote inserts data and returns it', async () => {
  const note = {
    content: 'this is my note',
    id: 1,
    tags: ['hello'],
  };

  // Tell the mock what to "pretend" the DB returned
  insertDB.mockResolvedValue(note);

  const result = await newNote(note.content, note.tags);

  // Only check the properties we actually have control over —
  // newNote generates its own `id`, so we can't assert on that
  expect(result.content).toEqual(note.content);
  expect(result.tags).toEqual(note.tags);
});
```

A couple of details worth understanding here:

- `insertDB.mockResolvedValue(note)` sets up the mock so that calling it returns a resolved Promise with `note` as the value — since the real `insertDB` is `async` and returns a Promise.
- **`toEqual` vs `toBe`**: `toBe` checks strict equality (basically `===`), which means two objects with identical properties but different memory addresses will _always_ fail `toBe` — objects are never `===` equal to each other even if their contents match. `toEqual` instead checks that two things have the same properties/values, which is what you want when comparing objects.
- The test above originally tried to assert the whole `result` object equals the whole `note` object, but that failed — because `newNote` generates its own `id` internally, so the ids won't match. The fix is to only assert on the fields you actually control (`content`, `tags`), not machine-generated ones like `id`.
- Because of the ESM setup, the Jest command itself has to run differently — the `test` script in `package.json` needs to look like:
    
    ```json
    "scripts": {  "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js"}
    ```
    
    This tells Node to run Jest with experimental support for ES modules, otherwise Jest doesn't understand the `import`/`export` syntax properly.

### On unit testing vs. behavior/user-focused testing

A student asked whether unit tests should be written from a technical point of view or from a user's perspective. The answer: it depends on the type of test. For unit tests, thinking about "the user" doesn't really make sense — you're testing a tiny, abstracted piece of logic that has no concept of a user. But for integration and end-to-end tests, thinking in terms of _behaviors_ makes much more sense, since at that level you're closer to what a real user actually does. This is also where terms like **BDD (behavior-driven development)** and **TDD (test-driven development)** come from — different philosophies about _how_ you approach writing tests, and different organizations lean toward different ones.

## Organizing Tests: `describe` blocks and `test` vs `it`

Once you have multiple tests for the same function (which is normal — a real app might have hundreds or thousands of tests), it helps to group related tests together using `describe`:

```javascript
describe('cli app', () => {
  test('newNote inserts data and returns it', async () => {
    // ...
  });

  test('getAllNotes returns notes from the database', async () => {
    // ...
  });

  test('removeNote returns undefined when the note does not exist', async () => {
    // ...
  });
});
```

This just visually nests/groups the tests together in the output, which makes large test suites much easier to scan.

You may also see `it(...)` used instead of `test(...)` in some codebases — they're functionally identical, just a stylistic choice. Some people prefer writing tests as readable sentences (`it('should return undefined when note is missing', ...)`), while others prefer the more direct `test('does X', ...)` style. Both behave the same way under the hood.

---

## Summary / Key Takeaways

- **Testing comes in layers**: unit (one small piece, isolated) → integration (a whole flow of functions working together) → end-to-end (a real user's full journey, usually needs a headless browser) → API testing (the full contract of one API call: status, headers, behavior).
- **Jest is the tool used here**: test files are conventionally named `*.test.js`, tests are run with `npm test`, and the core building blocks are `test(name, fn)` and `expect(value).matcher(...)`.
- **Mocking lets you isolate the unit under test** by faking out dependencies (like database calls) so you don't have to hit a real database — but doing this with ES Modules requires `jest.unstable_mockModule` plus dynamic `await import()`, which is more complex than the CommonJS equivalent.
- **`toEqual` checks matching properties/values; `toBe` checks strict identity** — two different objects with the same data will never pass `toBe`, so use `toEqual` when comparing objects, and only assert on fields you actually control (skip auto-generated ones like `id`).
- **Use `beforeEach` to reset mock state between tests**, and use `describe` blocks to group related tests together — both make larger test suites reliable and easier to navigate.