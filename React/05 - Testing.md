
## Executive Overview

Testing is the practice of writing code that verifies your application code behaves the way you expect. In the modern React ecosystem, testing sits at the intersection of three concerns: **confidence** (will this code keep working as I change the codebase around it?), **regression prevention** (once a bug is fixed, can I guarantee it never silently comes back?), and **documentation** (a well-named test describes what a piece of code is supposed to do).

This guide walks through a complete, practical testing setup for a modern React application built with **Vite**, using:

- **Vitest** — the test runner/harness built by the Vite team, designed as a drop-in replacement for Jest that reuses your existing Vite configuration.
- **@testing-library/react** — a library of helper utilities for rendering React components and querying the resulting DOM the way a _user_ would experience it, rather than the way a developer thinks about implementation details (CSS classes, internal state, etc.).
- **happy-dom** — a lightweight, fast, synthetic browser environment that runs inside Node.js so that tests don't need a real browser.
- **vitest-fetch-mock** — a helper for mocking `fetch` calls so tests don't hit real APIs.
- **@vitest/coverage-v8 / @vitest/coverage-istanbul** — tools for measuring how much of your source code is actually exercised by your tests.
- **@vitest/ui** — a browser-based dashboard for running and inspecting tests.
- **@vitest/browser + Playwright** — an experimental (as of this recording) way to run your component tests inside _real_ browsers (Firefox, Chromium, WebKit) instead of a simulated DOM.

By the end of this guide, you'll understand how to render components in isolation, simulate user interactions, mock network requests, test custom hooks, measure code coverage, use snapshot testing responsibly, and run tests inside real browsers — as well as the philosophy of _why_ you'd choose one testing strategy over another.

### A Philosophy of Testing (Setting Expectations)

Before diving into mechanics, it's worth internalizing the philosophy that shapes _how_ this guide approaches testing, since it directly informs which techniques are emphasized and which are treated with suspicion:

- **Fewer, high-value tests beat exhaustive low-value tests.** A test suite full of tests nobody trusts or understands is worse than no tests at all.
- **A good test fails fast, fails loudly, and tests something that matters.** If a test doesn't do all three, it's a candidate for deletion.
- **Deleting tests is healthy.** Test suites are living artifacts, not monuments — if a test is flaky or no longer valuable, remove it.
- **Integration-style tests that mimic real user flows** (e.g., "a user can log in and buy something") are extremely valuable, but you only need a handful of them — large end-to-end suites (historically built with tools like Selenium) tend to become slow, flaky, and full of false positives/negatives that eat up developer time.
- **100% test coverage is not a worthy goal.** Chasing a coverage percentage encourages writing tests for the sake of the metric rather than tests that protect something meaningful.
- **A great moment to introduce a test:** when you feel cognitive overwhelm about all the business rules and edge cases in your head. Writing a test externalizes that mental model and reduces anxiety about breaking something.
- **A great workflow for fixing bugs:** when given a bug report, first write a test that reproduces (and currently fails on) the bug, _then_ fix the code until the test passes. This gives you both a regression guard and confidence that the fix actually worked.

---

## 1. Environment Setup

### 1.1 The Tools and Why Each One Exists

|Package|Role|Mental Model|
|---|---|---|
|`vitest`|Test runner / harness|The engine that discovers, executes, and reports on your tests. Built by the Vite team so it automatically understands your `vite.config.js` — no separate configuration step required, unlike older tools (e.g., Mocha) where you had to wire everything by hand.|
|`@testing-library/react`|Component rendering + DOM querying utilities|Provides `render()`, and query methods (`getByRole`, `getByPlaceholderText`, `findByRole`, etc.) that push you toward testing _what a user perceives_, not _implementation details_. It is the spiritual successor to older, more brittle tools like Airbnb's **Enzyme**.|
|`happy-dom`|Synthetic DOM environment|A slimmed-down alternative to the older, heavier `jsdom`. It emulates a browser's DOM APIs inside Node.js so `render()` has something to render into, without needing an actual browser process. Faster, but less feature-complete — unusual/edge-case browser behavior may not be perfectly emulated.|
|`vitest-fetch-mock`|Fetch mocking helper|Wraps `vi` (Vitest's built-in mocking/spy library) to make faking `fetch()` responses ergonomic.|
|`@vitest/coverage-v8` / `@vitest/coverage-istanbul`|Code coverage instrumentation|Reports which lines/branches/functions of your source were actually executed during the test run.|
|`@vitest/ui`|Visual test dashboard|An in-browser UI for running, filtering, and inspecting tests, including a module dependency graph and per-test console output.|
|`@vitest/browser` + `playwright` + `vitest-browser-react`|Real-browser test execution (experimental)|Runs your component tests inside actual browser engines rather than a simulated DOM.|

**Historical lineage worth knowing:** Jasmine → Jest → Vitest. Jest (originally built by Facebook, now under the OpenJS Foundation as a fully open-source project) mimicked Jasmine's API style, and Vitest in turn mimics Jest's API style. This is why so much Jest knowledge (the `test()`/`expect()`/`describe()` API shape) transfers directly to Vitest with almost no changes.

### 1.2 Installing Dependencies

```bash
npm install -D vitest @testing-library/react happy-dom
```

Representative versions used in this walkthrough:

```
vitest@2.1.3
@testing-library/react@16.0.1
happy-dom@15.7.4
```

### 1.3 Configuring Vite for Testing

Vitest reads your existing `vite.config.js`, so in most cases there is almost nothing new to configure. You only need to tell it which DOM environment to simulate:

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'happy-dom',
  },
});
```

Then point the `test` script in `package.json` at Vitest:

```json
{
  "scripts": {
    "test": "vitest"
  }
}
```

Running it can be done any of these equivalent ways (in increasing order of laziness):

```bash
npm run test
npm test
npm t
```

> **Note on `vitest.config.js`:** If you ever need a _dedicated_ Vite config just for your test environment (separate from your app's build config), you can create a `vitest.config.js` file. Critically, you must explicitly extend your existing `vite.config.js` inside it (via `extends`), or Vitest will silently ignore your app's Vite configuration and you'll end up debugging confusing failures.

### 1.4 Where Tests Live

Vitest auto-discovers tests using two conventions, and you can use either (or both):

1. A directory named **`__tests__`** — the double-underscore naming convention is borrowed from Python (compare to `__init__.py`). The double underscores are a _signal_, not a stylistic choice: they mark the directory as having special meaning to the tooling, and renaming it will break test discovery.
2. Any file matching the pattern **`*.test.jsx`** or **`*.spec.jsx`** (e.g., `app.test.jsx`, `Pizza.spec.jsx`), placed anywhere in your source tree — including directly alongside the component it tests.

Either naming convention (`.test.` or `.spec.`) is functionally equivalent; it's a matter of team preference.

---

## 2. Writing Your First Component Test

### 2.1 Anatomy of a Basic Render Test

Consider a `Pizza` component that renders an image with `alt` text for accessibility. We want a test asserting that the alt text is always correctly set (imagine this being a legal/accessibility requirement).

```jsx
// __tests__/Pizza.test.jsx
import { render } from '@testing-library/react';
import { expect, test } from 'vitest';
import Pizza from '../Pizza';

test('alt text renders on Pizza images', async () => {
  const name = 'My favorite Pizza';
  const src = 'https://picsum.photos/200';

  const screen = render(
    <Pizza name={name} description="super cool pizza" image={src} />
  );

  const img = screen.getByRole('img');

  expect(img.src).toBe(src);
  expect(img.alt).toBe(name);
});
```

### 2.2 Breaking Down Each Piece

- **`test(name, fn)`** — declares a single test case. The string you provide is purely for humans: it's what shows up in the failure output, so it should be descriptive enough that you know _exactly_ what broke just by reading it (e.g., prefer `"alt text renders on Pizza images"` over a vague `""`).
- **`async` by default** — the instructor's habit is to make every test function `async` even when nothing inside it currently awaits anything. The reasoning: as soon as you introduce an asynchronous operation later (an API call, a `waitFor`, etc.) and forget to mark the function `async`, you get a confusing failure. Making it a habit up front costs nothing (Vitest handles both sync and async test functions transparently) and prevents a recurring class of "why is this failing" moments.
- **`render(...)`** — conceptually equivalent to calling `ReactDOM.createRoot(...).render(...)`, but performed entirely inside the synthetic `happy-dom` environment in Node.js. This is dramatically faster than spinning up a real browser for every test.
- **The returned `screen` object** — an instance of `RenderResult` from React Testing Library. It exposes a large family of query methods for finding elements the way a _user_ would identify them, rather than via implementation details like class names or component internals.

### 2.3 Why Query "By Role" Instead of By Class/ID

Testing Library deliberately biases its API toward **user-perceivable semantics**:

- `getByRole('img')`, `getByRole('button')`, `getByRole('heading')` — corresponds to ARIA roles a user (or assistive technology) would perceive.
- `getByPlaceholderText(...)` — corresponds to what a sighted user visually experiences in a form field before typing.
- `getByLabelText(...)`, `getByText(...)`, `getByTestId(...)` — each targets a different facet of what's observable to a real user (or, in the case of test IDs, an escape hatch when no other semantic hook is available).

**The mental shift:** a CSS class name is a developer concern. Users don't care whether `.card-header` exists — they care whether they can _see_ an image, _read_ a heading, or _click_ a button. By querying based on role/text/label, your tests stay valid even if you refactor your styling, and they naturally nudge you toward writing more accessible markup (since if `getByRole('img')` can't find your image, that's often a sign of missing semantic HTML or missing `alt` text in the first place).

### 2.4 Confirming a Test Actually Tests Something

A healthy habit: if a test passes on the very first run, be suspicious. Deliberately break the implementation (e.g., temporarily hardcode `alt={"lol no"}`) and confirm the test _fails_ with a clear, informative diff (expected vs. received). Only after confirming the test can fail should you trust that it passes for the right reason.

---

## 3. Cleaning Up Between Tests

### 3.1 The Problem: Testing Library's `screen` Is Stateful

If you render multiple components across multiple `test()` blocks without cleanup, the DOM accumulates content from _previous_ tests. This can cause a query like `getByRole('img')` to match a stale element from an earlier test rather than the one you just rendered — leading to baffling assertion failures where the "received" value looks like it came from an entirely different test case.

### 3.2 The Fix: `afterEach(cleanup)`

```jsx
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(cleanup);
```

- `cleanup` (from `@testing-library/react`) unmounts everything that was rendered and clears the DOM.
- `afterEach` (from `vitest`) is a lifecycle hook that runs the given callback after every individual test in the file.

**Why this plumbing is necessary at all:** `@testing-library/react` is a general-purpose library usable across multiple test runners (Vitest, Mocha, Jest, etc.). It has no built-in knowledge of when a "test" starts or ends in _your specific_ runner — that concept belongs to the runner, not the rendering library. So you must explicitly bridge the two by telling your runner's lifecycle hook to call the rendering library's cleanup function.

You could alternatively call `cleanup()` manually at the end of each test, but wiring it once via `afterEach` is far less error-prone and guarantees it always runs, even if a test throws.

---

## 4. Mocking Network Requests

### 4.1 Why You Should Never Hit a Real API in Tests

You technically _could_ spin up a real (or fake) backend server and let your components make real network calls during tests, but this is strongly discouraged — it introduces flakiness, slowness, and external dependencies into what should be fast, deterministic unit/integration tests. The recommended approach is to **mock** `fetch` so that a request appears to succeed (or fail) with data you control, without any network traffic actually occurring.

### 4.2 Setting Up `vitest-fetch-mock`

```bash
npm install -D vitest-fetch-mock@0.3.0
```

```jsx
import { createFetchMock } from 'vitest-fetch-mock';
import { vi } from 'vitest';

const fetchMocker = createFetchMock(vi);
fetchMocker.enableMocks();
```

- **`vi`** is Vitest's built-in spy/mocking library — conceptually the same role that libraries like **Sinon** played in older React testing setups. A "spy" is a fake stand-in for a real function that records how it was called (arguments, call count) so your test can later assert on that call history.
- **`createFetchMock(vi)`** wires `vitest-fetch-mock`'s mocking utilities into Vitest's spy system.
- **`fetchMocker.enableMocks()`** monkey-patches the global `fetch` so that, instead of making a real network request, it returns whatever response you've configured — while still letting you inspect exactly how it was called afterward.

### 4.3 Example: Testing a Form Submission with Mocked API Calls

Assume a `contact.lazy.jsx` route file (using TanStack Router's "lazy route" convention) that renders a contact form, and uses TanStack Query (`useMutation`) to `POST` the form data.

```jsx
// contact.lazy.test.jsx
import { render } from '@testing-library/react';
import { expect, test, vi } from 'vitest';
import { createFetchMock } from 'vitest-fetch-mock';
import { QueryClientProvider, QueryClient } from '@tanstack/react-query';
import { Route } from '../routes/contact.lazy';

const fetchMocker = createFetchMock(vi);
fetchMocker.enableMocks();

test('can submit contact form', async () => {
  fetchMocker.mockResponseOnce('', { status: 200 });

  const queryClient = new QueryClient();
  const screen = render(
    <QueryClientProvider client={queryClient}>
      <Route.options.component />
    </QueryClientProvider>
  );

  const nameInput = screen.getByPlaceholderText('Name');
  const emailInput = screen.getByPlaceholderText('Email');
  const msgTextArea = screen.getByPlaceholderText('Message');

  const testData = {
    name: 'Brian',
    email: 'dustinshatemail@example.com',
    message: 'Brian teach, Dustin.',
  };

  nameInput.value = testData.name;
  emailInput.value = testData.email;
  msgTextArea.value = testData.message;

  const btn = screen.getByRole('button');
  btn.click();

  const h3 = await screen.findByRole('heading', { level: 3 });
  expect(h3.innerText).toContain('Submitted');

  const requests = fetchMocker.requests();
  expect(requests.length).toBe(1);
  expect(requests[0].url).toBe('/api/contact');
  expect(fetchMocker).toHaveBeenCalledWith('/api/contact', {
    body: JSON.stringify(testData),
    headers: { 'Content-Type': 'application/json' },
    method: 'POST',
  });
});
```

### 4.4 Key Concepts in This Test

- **`fetchMocker.mockResponseOnce(body, opts)`** — configures the _next single_ call to `fetch` to resolve with the given body and status. Since this component only issues one API call, there's no need to distinguish between multiple mocked endpoints. (If a component made nine different API calls, you would need to differentiate responses per-route.)
- **Why a `QueryClientProvider` is needed in the test:** TanStack Query's hooks (like `useMutation`) require a `QueryClient` to be available via context. In the real app, this is provided once at the root (`App`). Since the test renders the route's component in isolation — not inside the full `App` tree — it must wrap the component in its own `QueryClientProvider` with a fresh `QueryClient` instance so the hook has something to talk to.
- **`Route.options.component`** — when using TanStack Router's file-based "lazy route" pattern, the default export of a route file isn't the bare component; it's a `Route` object with routing machinery attached (path matching, loaders, etc.). To render _just_ the visual component in a test (without the router), you reach into `Route.options.component`.
- **Simulating user input** — setting `.value` directly on the input DOM nodes retrieved via `getByPlaceholderText`, then calling `.click()` on the button retrieved via `getByRole('button')`, mimics a user typing into a form and submitting it.
- **`await screen.findByRole(...)`** — the `find*` family of queries (as opposed to `get*`) is **asynchronous and polling**: it repeatedly checks the DOM until either the queried element appears or a timeout is reached. This is essential for asserting on UI that appears only _after_ an async operation resolves (e.g., a mutation completing and the component transitioning from a loading state to a "Submitted!" state).
- **`toContain` vs. `toBe`** — using `toContain` for a fuzzy substring match (e.g., checking the text contains `"Submitted"`) is more resilient than an exact `toBe` match, which would break if a minor wording change (like adding an exclamation point) occurred.
- **Testing the API call itself is a deliberate choice, not an accident.** Some testers consider asserting on the exact request body/headers/method to be "testing implementation details" and prefer to avoid it. The counter-argument presented here: if the wrong data (or no data) is sent to the server, the user's submission is effectively lost — and that is a _user-facing_ consequence, which justifies testing it even though it dips into some implementation-level detail.

### 4.5 A Common Gotcha

A subtle typo bug surfaced during this walkthrough: passing a prop named `queryClient` to `<QueryClientProvider>` when the component actually expects a prop named `client`. This kind of prop-name mismatch is exactly the class of bug that **TypeScript** would catch immediately at compile time (since `QueryClientProvider`'s prop types would flag the unexpected/missing prop) — a good illustration of one of the practical benefits of typed React code even outside of testing.

---

## 5. Testing Custom Hooks

### 5.1 Why Test Hooks Directly?

Custom hooks are prime candidates for focused testing because:

- They are often reused across dozens of components — a bug in the hook silently propagates everywhere it's used.
- Testing the hook in isolation gives you confidence in the underlying logic without needing to render (and reason about) every component that consumes it.

A candid observation from the walkthrough: hook tests often end up **longer than the hook's own implementation** — this is normal and not a sign you're doing something wrong.

### 5.2 The Problem: Hooks Can't Be Called Outside a Component

React hooks rely on React's internal call-order tracking, which only exists during a component's render. You cannot simply call `usePizzaOfTheDay()` as a bare function in a test file. To exercise a hook, you need _some_ component rendering it.

### 5.3 The Manual (Educational) Approach

To understand what's really happening under the hood, you can build this scaffolding by hand:

```jsx
let pizza;

function TestComponent() {
  pizza = usePizzaOfTheDay();
  return null;
}

function getPizzaOfTheDay() {
  render(<TestComponent />);
  return pizza;
}
```

- `TestComponent` renders `null` — its only job is to invoke the hook as a side effect and stash the result in an outer-scope variable.
- `getPizzaOfTheDay()` renders that throwaway component (the actual rendered output is discarded — it's always `null`) purely to trigger the hook call, then hands back whatever the hook produced.

This demystifies the "magic" of hook-testing utilities: they are not doing anything exotic, they're doing exactly this — wrapping your hook in an invisible component, rendering it once, and handing you the result.

### 5.4 The Idiomatic Approach: `renderHook`

React Testing Library ships a purpose-built utility, `renderHook`, that performs the exact pattern above for you:

```jsx
import { renderHook } from '@testing-library/react';
import { expect, test, vi } from 'vitest';
import { createFetchMock } from 'vitest-fetch-mock';
import { usePizzaOfTheDay } from '../usePizzaOfTheDay';

const fetchMocker = createFetchMock(vi);
fetchMocker.enableMocks();

const testPizza = {
  name: 'The calabrese pizza',
  category: 'Supreme',
  description: 'lol pizza from Calabria',
  image: '/public/pizzas/calabrese.webp',
  sizes: { small: 12.25, medium: 16.25, large: 20.25 },
};

test('gives null when first called', async () => {
  fetchMocker.mockResponseOnce(JSON.stringify(testPizza));

  const result = renderHook(usePizzaOfTheDay);

  expect(result.current).toBe(null);
});
```

- **`renderHook(fn)`** takes the hook function itself and internally creates and renders the equivalent of `TestComponent`, then returns a `result` object.
- **`result.current`** always reflects the _most recent_ value returned by the hook. On the very first render — before any `useEffect` inside the hook has had a chance to fire and update state — the hook understandably returns `null` (its initial state), since data-fetching hooks typically start in an "unloaded" state and populate asynchronously.

### 5.5 Waiting for Async State Updates: `waitFor`

Once the hook's internal `useEffect` fires, makes a fetch call, and updates state, `result.current` will eventually reflect the fetched data — but not synchronously. Use `waitFor` to poll until that happens:

```jsx
import { waitFor } from '@testing-library/react';

test('calls the API and gives back the pizza of the day', async () => {
  fetchMocker.mockResponseOnce(JSON.stringify(testPizza));

  const result = renderHook(usePizzaOfTheDay);

  await waitFor(() => expect(result.current).toEqual(testPizza));
  expect(fetchMocker).toBeCalledWith('/api/pizza-of-the-day');
});
```

**How `waitFor` actually works:** you give it a callback containing one or more `expect(...)` assertions. `waitFor` calls that callback repeatedly, at short intervals, and simply keeps retrying **as long as the callback keeps throwing an error** (which is exactly what a failing `expect` does — it throws). The moment the callback runs without throwing, `waitFor` resolves successfully. If a timeout is reached (several seconds by default) without the assertion ever passing, the test fails with the last thrown error. This makes `waitFor` ideal for "eventually consistent" UI states driven by asynchronous effects, without resorting to arbitrary fixed `setTimeout` delays.

### 5.6 Test Coverage Discipline for Hooks

A well-tested hook, at minimum, should cover:

1. **The initial/loading state** (e.g., returns `null` before data arrives).
2. **The happy path** (correct data eventually arrives and the correct API endpoint was called).
3. **Error/edge cases** — e.g., what happens if the API returns nothing for "today," or the fetch fails outright. These are called out as valuable follow-up tests even when time constraints mean they aren't written in the initial pass.

---

## 6. Snapshot Testing

### 6.1 What Snapshot Testing Actually Does

A snapshot test renders a component, serializes its output to a string (or fragment), and compares that output against a previously saved reference ("the snapshot"). If the two don't match, the test fails, forcing you to either:

- **Investigate** — did something break unintentionally?
- **Update the snapshot** — was this an intentional, expected change to the UI?

```jsx
// cart.test.jsx
import { expect, test } from 'vitest';
import { render } from '@testing-library/react';
import Cart from '../Cart';

test('snapshot with nothing in cart', () => {
  const asFragment = render(<Cart />).asFragment;
  expect(asFragment()).toMatchSnapshot();
});
```

On first run, Vitest creates a `__snapshots__` directory containing a generated file with the serialized markup (e.g., the rendered `<div class="cart">...</div>` structure). **This snapshot file should be committed to version control**, so that the expected output is durable and comparable across commits, not just across local test runs.

### 6.2 Updating Snapshots

If a change to the rendered output is _intentional_ (e.g., you deliberately restyled the cart), Vitest's watch mode lets you press **`u`** to update the stored snapshot to match the new output — effectively saying "yes, this new markup is now correct, remember it going forward."

### 6.3 The Case Against Overusing Snapshots

The instructor is candid about being skeptical of snapshot testing in general, for several concrete reasons:

- **It's extremely low effort but also low confidence.** The entire test is often just two lines, but it doesn't actually assert anything meaningful about _behavior_ — it only asserts that markup hasn't changed shape, character for character.
- **It's brittle by nature.** Any incidental change — even a harmless typo fix or minor style tweak — fails the test, training developers to reflexively hit "update" without carefully reviewing the diff. Once that habit sets in, the test stops providing real signal.
- **Snapshots compose badly with nested components.** If a low-level component seven layers deep changes its output, _every_ snapshot test for every ancestor component that renders it (directly or indirectly) will also fail, creating cascades of unrelated-looking failures from one small change.
- **UI changes constantly during active development**, so snapshot tests of frequently-iterated UI produce a lot of noise relative to the confidence they provide.

### 6.4 Where Snapshot Testing _Does_ Make Sense

- **Stable, "dumb" display components** that purely render data with no significant logic (a cart summary, a header, a footer, a nav bar) — components not expected to change shape often.
- **API contract shape verification.** Snapshot testing is not limited to React components — you can snapshot _any_ JavaScript object. If a backend service returns a specific object shape, snapshotting that shape (`expect(apiResult).toMatchSnapshot()`) is a lightweight way to catch accidental, unintended changes to the frontend/backend data contract, helping the two stay in sync over time. This use case earns more genuine endorsement than snapshotting arbitrary UI.

### 6.5 Inline Snapshots

Vitest also supports **inline snapshots**, where the expected value is written directly into the test file itself (rather than a separate `__snapshots__` file) the first time the test runs:

```jsx
expect(someValue).toMatchInlineSnapshot();
```

This is a stylistic alternative some teams prefer over managing separate snapshot files, though the instructor personally favors the external-file approach.

---

## 7. Code Coverage

### 7.1 What Coverage Measures

Code coverage reports tell you, line-by-line, branch-by-branch, and function-by-function, whether your test suite actually executed that piece of code at least once. It answers "what percentage of my source code do my tests even _touch_?" — not whether the code is _correct_.

### 7.2 Setting Up Coverage (V8 Provider)

```bash
npm install -D @vitest/coverage-v8@2.1.3
```

```js
// vite.config.js
export default defineConfig({
  test: {
    environment: 'happy-dom',
    coverage: {
      reporter: ['text', 'json', 'html'],
    },
  },
});
```

```json
{
  "scripts": {
    "coverage": "vitest --coverage"
  }
}
```

```bash
npm run coverage
```

- **`text`** reporter prints a table directly to your terminal.
- **`json`** reporter is useful for ingesting into external dashboards or CI tooling.
- **`html`** reporter generates a browsable static site (open `coverage/index.html`) with a fully interactive, file-by-file, line-by-line breakdown of exactly which lines executed and how many times.

### 7.3 V8 vs. Istanbul

- **V8** — the newer coverage provider, using V8's own built-in instrumentation. Generally recommended as the default choice.
- **Istanbul** — the long-standing, historically dominant coverage tool ("the gold standard" for a long time), still fully supported and sometimes required for compatibility with certain environments (notably, coverage inside **Playwright-based browser tests currently requires Istanbul**, since V8 coverage does not work with Playwright).

```bash
npm install -D @vitest/coverage-istanbul
```

```js
coverage: {
  provider: 'istanbul',
  reporter: ['text', 'json', 'html'],
}
```

### 7.4 Reading a Coverage Report

The HTML report highlights, for each source file:

- Lines executed once vs. multiple times (helpful for spotting hot paths or unused branches).
- Lines **never executed at all** — e.g., a `cart.map(...)` loop that never ran because every test happened to use an empty cart, meaning the "cart with items" rendering path was never actually exercised. Similarly, an error-handling branch in an API-posting function might show as untested if no test ever forced that API call to fail.
- Aggregate percentages across **statements**, **branches**, and **functions** for the whole codebase.

### 7.5 Why 100% Coverage Isn't the Goal

Chasing 100% coverage is described as **a fool's errand** and, more strongly, "a waste of time" — because it drives you to write tests for code paths you don't actually care about protecting. **A test you don't care about is worse than no test at all**, because it adds maintenance burden (it must still be updated/kept passing as code evolves) without providing corresponding confidence. Coverage numbers are best used as a _diagnostic signal_ ("huh, this important branch is never tested — should it be?") rather than a target to maximize.

---

## 8. The Vitest UI

### 8.1 Setup

```bash
npm install -D @vitest/ui@2.1.3
```

```json
{
  "scripts": {
    "test:ui": "vitest --ui"
  }
}
```

```bash
npm run test:ui
```

### 8.2 What It Provides

An in-browser dashboard that shows:

- All discovered tests, their pass/fail status, and execution time.
- The ability to filter to only failing tests, or run individual tests on demand.
- A **module dependency graph**, visualizing what each test file imports/depends on.
- The underlying source code for each test alongside its results.
- Any `console.log` output produced during a given test run.

This is a purely ergonomic/inspection tool — it doesn't change how tests are written or what they assert, but it can make debugging failures and exploring a large suite considerably faster than parsing raw CLI output.

### 8.3 The Vitest VS Code Extension

Installing the official **Vitest** extension in VS Code adds a dedicated testing panel to the sidebar, letting you:

- See and run all tests directly from the editor's test explorer.
- Run a single test with one click and jump straight to the exact line/reason for a failure.
- Enable **continuous run** mode, so tests automatically re-run every time you save a file — and Vitest is smart enough to only re-run the tests relevant to the file you changed, not the entire suite.

> **Troubleshooting tip:** if the extension's testing panel doesn't appear (e.g., it silently asks for unrelated Python tooling), try disabling and reinstalling the extension, or run **"Reload Window"** via the command palette (`Cmd+Shift+P` / `Ctrl+Shift+P`) — a fix that resolves a surprisingly large fraction of general VS Code glitches.

**Choosing your workflow:** there is no single "correct" way to run tests day-to-day — CLI, the Vitest UI, and the VS Code extension all surface the same underlying test results. Pick whichever fits your existing workflow; a developer who lives in VS Code all day will naturally lean on the extension or CLI over the web UI, and vice versa.

---

## 9. Running Tests in Real Browsers (Experimental)

### 9.1 Motivation and Status

Everything covered so far runs inside **happy-dom**, a _simulated_ browser environment inside Node.js. This is fast, but by definition an approximation — genuinely unusual browser behaviors can occasionally behave differently (or not be supported at all) compared to a real browser engine.

Vitest has an experimental browser mode that runs your tests inside **actual** browser engines via **Playwright** (Microsoft's browser automation tool, created by the same engineer, Pavel, who originally built **Puppeteer** at Google before moving to Microsoft). At the time of this walkthrough, this mode is explicitly called out as **not yet stable enough to fully recommend** for production use — the supporting library `vitest-browser-react` was at a `0.0.1`, clearly experimental version, likely to change API shape before it stabilizes.

### 9.2 Installing Browser-Mode Dependencies

```bash
npm install -D @vitest/browser@2.1.3 playwright vitest-browser-react
```

### 9.3 Configuring Multiple Test "Workspaces"

Vitest's `defineWorkspace` feature is intended primarily for **monorepos** (multiple packages in one repository, each with its own test configuration). This walkthrough repurposes it to run two _different kinds_ of tests side-by-side in a single project: existing happy-dom tests, and new real-browser tests — without needing to migrate every existing test to the browser immediately.

```js
// vitest.config.js
import { defineWorkspace } from 'vitest/config';

export default defineWorkspace([
  {
    extends: './vite.config.js',
    test: {
      include: ['**/*.node.test.{js,jsx}'],
      name: 'happy-dom',
      environment: 'happy-dom',
    },
  },
  {
    extends: './vite.config.js',
    test: {
      setupFiles: ['vitest-browser-react'],
      include: ['**/*.browser.test.{js,jsx}'],
      name: 'browser',
      browser: {
        provider: 'playwright',
        enabled: true,
        name: 'firefox', // or 'chromium' / 'webkit'
      },
    },
  },
]);
```

Key points:

- Each workspace entry **must** `extends` the base `vite.config.js`, or it will not inherit your app's existing Vite/React plugin setup.
- Tests are routed to the correct environment purely by **filename convention**: files ending in `.node.test.jsx` run under happy-dom; files ending in `.browser.test.jsx` run under Playwright. (You could instead separate them into different directories — the filename-suffix convention is a stylistic choice, informally described as "chaos-driven development.")
- The `name` field (`'happy-dom'` vs `'browser'`) is purely a display label, so that test output clearly indicates which environment a given test actually ran under.
- **`provider`** can be `'playwright'` (recommended) or `'webdriver'` (i.e., **Selenium**, listed as the other supported option).
- **`browser.name`** selects the actual engine: `'firefox'`, `'chromium'`, or `'webkit'` (for Safari-equivalent testing on macOS).

Existing test files need to be renamed to fit the new convention, e.g.:

```
contact.lazy.test.jsx    →  contact.lazy.node.test.jsx
usePizzaOfTheDay.test.jsx → usePizzaOfTheDay.node.test.jsx
```

Tests that don't strictly depend on happy-dom-specific behavior (such as the `cart` snapshot test — since Playwright supports snapshot testing natively) can instead be moved to the browser workspace:

```
cart.test.jsx  →  cart.browser.test.jsx
```

### 9.4 Writing a Browser-Mode Test

`vitest-browser-react` is designed to closely mirror the API of `@testing-library/react`, so migrating a test is largely mechanical:

```jsx
// pizza.browser.test.jsx
import { render } from 'vitest-browser-react';
import { expect, test } from 'vitest';
import Pizza from '../src/Pizza';

test('alt text renders on image', async () => {
  const name = 'My favorite pizza';
  const src = 'https://picsum.photos/200';

  const screen = render(
    <Pizza name={name} image={src} description="cool browser stuff" />
  );

  const image = await screen.getByRole('img');

  await expect.element(image).toBeInTheDocument();
  await expect.element(image).toHaveAttribute('src', src);
  await expect.element(image).toHaveAttribute('alt', name);
});
```

Notable differences from the happy-dom version of the same test:

- Because a real, external browser process is being driven, queries and assertions become genuinely **asynchronous** (`await screen.getByRole(...)`, `await expect.element(...)`) — you are actually reading from a live DOM inside a real Firefox/Chromium/WebKit process, not synchronously inspecting an in-memory simulated document.
- `expect.element(...)` is the browser-mode equivalent of directly asserting on properties like `img.src`/`img.alt` — it wraps the element reference so assertions can be awaited.
- Since `vitest-browser-react` aims for API parity with Testing Library, existing Testing Library documentation is a reasonable reference even for browser-mode-specific questions.

### 9.5 First-Time Setup Requirements

The first time you run Playwright-backed tests, you may need to install the actual browser binaries it drives:

```bash
npx playwright install
```

This downloads slimmed-down local copies of Firefox, Chromium, and WebKit (a few hundred megabytes total) that Playwright controls programmatically.

### 9.6 Practical Observations About Browser Mode

- Because it's driving a _real_ browser, you can literally watch the component render live — including images and CSS — which produces a side benefit: it can double as a lightweight, ad hoc "storybook"-style way to visually inspect individual components in isolation.
- The tooling is described as still somewhat fragile at this stage ("this breaks constantly... it will freeze") — usually resolved with a simple refresh, but a clear signal that this feature is not yet production-hardened.
- **Coverage caveat:** Vitest's default **V8** coverage provider does **not** work with Playwright-driven browser tests. If you need coverage numbers for browser-mode tests, you must switch your coverage provider to **Istanbul**, and additionally pin your browser workspace to a browser Istanbul supports well (Chromium, in this walkthrough) rather than Firefox.

---

## Common Pitfalls & Best Practices

|Pitfall|Why It Happens|How to Avoid It|
|---|---|---|
|**Tests bleed state into each other** (a later test sees leftover DOM from an earlier one)|`@testing-library/react`'s `render()` doesn't automatically unmount between tests, and the library has no innate awareness of your test runner's lifecycle.|Always add `afterEach(cleanup)` using `cleanup` from `@testing-library/react` and `afterEach` from `vitest`.|
|**Querying by CSS class or DOM structure instead of role/text**|Feels natural to developers used to jQuery-style selectors.|Prefer `getByRole`, `getByPlaceholderText`, `getByText`, `getByLabelText` — they mirror what a real user perceives and are resilient to internal markup/styling refactors. Fall back to `getByTestId` only when no semantic query fits.|
|**Forgetting `await` on `findBy*`/`waitFor`**|These queries are asynchronous by design (they poll), but it's easy to forget in a hurry.|Get in the habit of writing test functions as `async` by default, and always `await` anything that might resolve after a tick — asynchronous UI updates (loading → loaded states) require it.|
|**Testing implementation details versus user-facing behavior**|It can feel more "thorough" to assert on internal function calls, exact request bodies, or internal state.|Bias toward asserting what the user would observe (DOM text, visible elements). Reach for implementation-level assertions (like exact fetch call arguments) only when the implementation detail has real user-facing consequences (e.g., data being sent to a server correctly).|
|**Chasing 100% test coverage**|Coverage percentages are an easy, visible metric to optimize.|Treat coverage as a diagnostic tool for spotting _forgotten_ code paths, not a target. A meaningless test that only exists to bump coverage adds maintenance cost without adding confidence.|
|**Over-relying on snapshot tests**|They are extremely fast to write (as little as two lines) and give a false sense of "100% covered."|Reserve snapshots for stable, low-logic display components or for verifying API/object _shape_ contracts. Avoid snapshotting deeply nested or frequently-changing UI, since a single low-level change can cascade into many unrelated-looking failures.|
|**Not resetting mocked `fetch` responses between tests**|Mock state (like `mockResponseOnce`) can leak or be misconfigured across multiple test cases in the same file.|Configure fresh mock responses per test (`mockResponseOnce` inside each `test()` block), and keep one `QueryClient` instance scoped per test where relevant.|
|**Prop name typos silently breaking hook-dependent components** (e.g., passing `queryClient` when a component expects `client`)|Plain JavaScript/JSX won't catch mismatched prop names at compile time.|Adopt TypeScript for prop-type safety where feasible; in JS-only codebases, let failing tests (like a "no QueryClient set" runtime error) guide you directly to the mismatch.|
|**Building large, brittle end-to-end suites (e.g., heavy Selenium suites)**|It's tempting to test every user flow exhaustively at the UI level.|Keep a small number of high-value integration tests covering critical user journeys; rely on more numerous, faster, more isolated component/hook tests for everything else.|
|**Testing hooks by calling them directly**|Hooks rely on React's render cycle and cannot be invoked as bare functions outside a component.|Use `renderHook` (or the manual dummy-component pattern, if you want to understand what's happening under the hood) to safely exercise a hook in isolation.|

---

## Summary of Key Takeaways

- **Vitest** is a Vite-native test runner that mimics Jest's API (which itself mimicked Jasmine's), so it reads your existing `vite.config.js` and needs almost no separate setup — just specify a `test.environment` (e.g., `happy-dom`).
- **`@testing-library/react`** encourages querying the DOM the way a real user experiences it (`getByRole`, `getByPlaceholderText`, etc.) rather than via CSS classes or internal implementation details.
- **Always clean up between tests** with `afterEach(cleanup)` — Testing Library has no built-in awareness of your test runner's lifecycle.
- **Mock network requests** with `vitest-fetch-mock` + Vitest's built-in `vi` spy library; never let tests hit real APIs.
- **`renderHook`** is the standard way to test custom hooks in isolation; **`waitFor`** lets you assert on state that updates asynchronously by repeatedly retrying an assertion until it stops throwing.
- **Snapshot testing** is low-effort but low-confidence — reserve it for stable display components or for pinning down API/object shape contracts; avoid it for frequently-changing or deeply nested UI.
- **Code coverage** (via `@vitest/coverage-v8` or `-istanbul`) is a diagnostic tool for spotting untested code paths — not a goal to maximize. 100% coverage is explicitly discouraged as a target.
- **`@vitest/ui`** and the **Vitest VS Code extension** provide visual, interactive ways to run and debug tests, including continuous re-run on save.
- **Real-browser testing** via `@vitest/browser` + Playwright is powerful (and doubles as a live component previewer) but was, at the time of this material, still experimental — worth knowing about, not yet a default recommendation. Note also that V8 coverage doesn't work with Playwright; Istanbul is required if you need coverage there.
- **Philosophically:** favor a small number of tests you genuinely trust and care about over exhaustive suites optimized for a coverage number. A test that doesn't tell you anything useful is worse than no test at all — and deleting stale or low-value tests is a healthy, ongoing practice.