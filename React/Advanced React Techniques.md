
## Executive Overview

This guide covers a set of React patterns that sit outside the "everyday" toolkit of `useState`, `useEffect`, and controlled inputs. They are tools you won't reach for on every feature, but that you need to have in your back pocket because they solve problems that ordinary component composition cannot solve cleanly.

Specifically, this guide covers:

- **Portals** — rendering a component's output into a DOM node that lives _outside_ your React root, while keeping it logically inside your component tree (used for modals, tooltips, and "render into this other part of the page" scenarios).
- **Refs (`useRef`)** — a mutable, render-stable container used to hold values (often DOM nodes) that must **persist across renders** without themselves triggering a re-render.
- **`useEffect` cleanup functions** — the mechanism for preventing memory leaks and stale side effects when a component unmounts or re-runs an effect.
- **Custom Hooks vs. plain utility functions** — knowing when a piece of reusable logic should be a hook and when it's just a function.
- **Data fetching with TanStack Query** (`useQuery` / `useMutation`) — including conditional queries (`enabled`), query keys, caching, and how mutations differ from queries.
- **Class Components & Error Boundaries** — the one remaining use case (as of this writing) where a class component is required instead of a function component, plus the legacy lifecycle methods and `this`-binding quirks you'll encounter in older codebases.
- **Uncontrolled Forms** — using the native browser `FormData` API instead of tracking every keystroke in state, and why this is usually the _better_ default over controlled forms.

These techniques matter because modern React applications are built almost entirely from function components and hooks — but the ecosystem (libraries, legacy code, and specific edge cases like error handling) still requires you to understand the underlying class-based model and lower-level DOM-escape-hatch APIs like portals and refs. Mastering these fills in the last 10% of React knowledge that separates "I can build a to-do app" from "I can work in a large, real-world, years-old React codebase."

---

## 1. Portals

### 1.1 The Problem Portals Solve

A React application is normally mounted into a **single root DOM node**:

```html
<body>
  <div id="root"></div>
</body>
```

Everything your app renders is a descendant of that `#root` div. This is fine for 95% of UI, but it breaks down for **overlay UI** — modals, tooltips, dropdown menus, toasts — because of a fundamental problem: **CSS stacking context and DOM nesting**.

If a modal is rendered deep inside a component tree (e.g., inside a `<table>` inside a `<div style="overflow:hidden">`), it inherits the CSS constraints of its ancestors — clipped overflow, `z-index` stacking contexts, `transform`/`position` side effects, etc. Historically, developers fought this with:

- Extreme `z-index` values ("z-index wars"),
- Manually restructuring markup so overlay elements sit at the top of the DOM,
- Or trying to hack around it using React Context to "teleport" JSX (which only solves the _data_ problem, not the _DOM placement_ problem).

**Portals** solve this directly: they let a component render its children into a _different_ DOM node than the one where the component is logically defined in the tree — while still preserving React's normal event bubbling, context, and lifecycle behavior. The component's _logical_ position in the React tree doesn't change (it still receives props/context from its real parent) — only where its **actual rendered DOM output** appears.

> **Mental model:** Think of a portal as "render here in the tree, but _paint_ over there in the DOM."

### 1.2 A Real-World Use Case Beyond Modals

Portals aren't only for modals. A common pattern seen on documentation sites (e.g., cloud-provider docs) is a **contextual sidebar** that shows content related to whatever's on the main page. The sidebar and the main content are two structurally separate regions of the page, but they need to be driven by the _same_ React component logic. A portal lets a component that's logically part of your main page content render its output into that separate sidebar `<div>`.

### 1.3 Building a Reusable `Modal` Component

**Step 1 — Add a target node in your static HTML**, _outside_ (as a sibling to) your React root:

```html
<body>
  <div id="modal"></div>
  <div id="root"></div>
</body>
```

Note it's intentionally left empty — it should render nothing unless something is portaled into it.

**Step 2 — Build the `Modal` component:**

```jsx
import { useEffect, useRef } from "react";
import { createPortal } from "react-dom";

function Modal({ children }) {
  const elRef = useRef(null);
  if (!elRef.current) {
    elRef.current = document.createElement("div");
  }

  useEffect(() => {
    const modalRoot = document.getElementById("modal");
    modalRoot.appendChild(elRef.current);

    return () => {
      modalRoot.removeChild(elRef.current);
    };
  }, []);

  return createPortal(children, elRef.current);
}

export default Modal;
```

### 1.4 Deep Dive: Why `useRef` Instead of Creating the Div Inline?

This is the trickiest part of the pattern, so let's unpack it piece by piece.

- **The problem:** Every time a component re-renders, its function body runs again. If you wrote `const el = document.createElement("div")` directly in the render body, you would create a **brand-new DOM element on every single render** — which defeats the purpose, since the portal needs to keep injecting content into the _same_ underlying node.
- **Why not `document.getElementById` on every render?** Directly querying/mutating the DOM inside the render path is a **slow, imperative escape hatch**. React's whole rendering model is optimized around _not_ touching the real DOM more than necessary; doing manual DOM lookups on every render fights against that and is a performance anti-pattern.
- **The solution — `useRef`:**

```jsx
const elRef = useRef(null);
```

- `useRef(initialValue)` returns a **plain, mutable object** shaped like `{ current: initialValue }`.
- Critically, **this same object instance is preserved across re-renders**. React does not recreate it — it hands you back the exact same object reference every time the component re-renders (as long as the component doesn't unmount).
- Unlike `useState`, **mutating `ref.current` does NOT trigger a re-render.** This is by design: refs are an escape hatch for values that need to persist "outside" the render/state cycle — DOM nodes, timers, previous values, mutable instance variables, etc.
- The pattern `if (!elRef.current) { elRef.current = document.createElement("div") }` guarantees the `<div>` is created exactly **once** — on the very first render — and every subsequent render reuses that same object because `elRef.current` is now truthy and the condition is skipped.

**Execution flow:**

```
Render 1: elRef.current is null → create <div> → elRef.current = <div A>
Render 2: elRef.current is <div A> (truthy) → skip creation → still <div A>
Render 3: elRef.current is <div A> (truthy) → skip creation → still <div A>
...
```

This guarantees: **one component instance → one stable DOM node → reused across every render.**

### 1.5 Deep Dive: Why the DOM Insertion Happens in `useEffect`

```jsx
useEffect(() => {
  const modalRoot = document.getElementById("modal");
  modalRoot.appendChild(elRef.current);

  return () => {
    modalRoot.removeChild(elRef.current);
  };
}, []);
```

- **Why `useEffect` and not directly in the render body?** Effects are explicitly designed to run **after** React has committed changes to the DOM, as a place for side effects that touch the outside world (the DOM, subscriptions, timers, network calls). Appending a node to the document is exactly that kind of "outside world" side effect — it should not happen during the pure "calculate what the UI should look like" phase of rendering.
- **Why an empty dependency array `[]`?** This tells React: _"run this effect exactly once, right after the initial mount, and never again (until unmount)."_ Since we only need to insert the div into the DOM one time, this is correct — we don't want to re-insert it on every re-render.

### 1.6 Deep Dive: The Cleanup Function (and the Memory Leak It Prevents)

This is one of the most important lessons in the whole pattern. Consider what happens **without** a cleanup function:

- Every time the `Modal` component **mounts** (e.g., the user opens it), a new `<div>` gets appended to `#modal`.
- Every time it **unmounts** (e.g., the user closes it), **nothing removes that div** — it just silently stays in the DOM forever.
- If a user opens and closes the modal 1,000 times, you now have **1,000 orphaned, empty `<div>` elements** sitting inside `#modal`, invisible but consuming memory and bloating the DOM tree indefinitely. This is a textbook **memory leak**.

The fix: whatever function you `return` from inside a `useEffect` callback is treated by React as a **cleanup function**. React guarantees to call it:

1. Before the effect re-runs (if dependencies change), and
2. When the component **unmounts**.

```jsx
return () => {
  modalRoot.removeChild(elRef.current);
};
```

Since our dependency array is `[]`, this cleanup only fires once — when the `Modal` component is removed from the tree — and it removes exactly the one div this instance created. This closes the leak: **one mount → one insert; one unmount → one remove.**

> **General rule:** Any time an effect creates a subscription, timer, event listener, or manually attaches something to the DOM/an external system, ask yourself: _"What has to be undone when this component goes away?"_ That undo logic belongs in the effect's cleanup function.

### 1.7 Understanding `children` and `createPortal`

```jsx
function Modal({ children }) {
  // ...
  return createPortal(children, elRef.current);
}
```

- `children` is a **special, reserved prop name** in React. Whatever JSX you nest _between_ a component's opening and closing tags becomes available to that component as `props.children` (here destructured directly out of props as `{ children }`).

```jsx
<Modal>
  <h1>Hello</h1>
</Modal>
```

Inside `Modal`, `children` will be the `<h1>Hello</h1>` element.

- This is exactly analogous to how a plain HTML element's children work — `<div><h1>Hi</h1></div>` means the `<h1>` is the "child content" of the `div`. Not every component needs children (many are "self-closing," e.g. `<Pizza />`, because there's nothing meaningful to nest inside them) — but any component meant to **wrap** arbitrary content, like `Modal`, needs to accept and render `children`.
- `createPortal(children, domNode)` is the React DOM API that does the actual teleportation: it tells React, _"render this `children` output, but attach the resulting DOM nodes under `domNode` instead of under this component's normal DOM parent."_

**Usage — anywhere in your app:**

```jsx
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div>
      <button onClick={() => setShowModal(true)}>Open</button>
      {showModal && (
        <Modal>
          <h2>I appear over everything else!</h2>
          <button onClick={() => setShowModal(false)}>Close</button>
        </Modal>
      )}
    </div>
  );
}
```

Even though `<Modal>` is written deep inside `App`'s JSX, its actual DOM output is physically attached under `#modal`, sitting as a sibling to `#root` — completely free of any clipping/z-index issues from `App`'s ancestor elements.

---

## 2. Conditional Rendering Patterns (Modals in Practice)

A very common real-world pattern: track "what is currently selected/focused" in state, and derive whether to show a modal from that state, rather than tracking a separate boolean.

```jsx
const [focusedOrder, setFocusedOrder] = useState();
```

- `focusedOrder` starts as `undefined` (no argument passed to `useState()`).
- Clicking a row sets it: `onClick={() => setFocusedOrder(order.order_id)}`.
- Closing/deselecting it just resets it to something falsy: `onClick={() => setFocusedOrder(null)}` (or `undefined`, or `0`).

Rendering logic:

```jsx
{focusedOrder ? (
  <Modal>
    {/* modal contents based on focusedOrder */}
  </Modal>
) : null}
```

- **Why `null` and not an empty fragment (`<></>`)?** Functionally, both work — `null`, `undefined`, `false`, and empty fragments all tell React "render nothing here." The stylistic argument for `null` is **explicitness**: it directly communicates "I want literally nothing here," rather than "I want an empty grouping wrapper." This is a matter of team convention rather than a hard technical rule.

### Guarding Data Fetches Behind Conditional State

A common gotcha: if you derive a data-fetch from a piece of state that might not exist yet (like `focusedOrder` before anything's been clicked), an unconditional fetch will fire immediately with a garbage/undefined value. TanStack Query's `useQuery` solves this with an `enabled` option:

```jsx
const {
  isLoading: isLoadingPastOrder,
  data: pastOrderData,
} = useQuery({
  queryKey: ["pastOrder", focusedOrder],
  queryFn: () => getPastOrder(focusedOrder),
  staleTime: 1000 * 60 * 60 * 24, // one day, in milliseconds
  enabled: !!focusedOrder,
});
```

Breaking this down:

- **`queryKey`** — a unique array TanStack Query uses to cache and de-duplicate requests. Much like React's `key` prop for lists, it needs to uniquely identify _this specific piece of data_. Including `focusedOrder` in the key means each distinct order gets its own cache entry — switching between orders doesn't require re-fetching an order you've already loaded.
- **`queryFn`** — the actual async function that fetches the data.
- **`staleTime`** — how long (in ms) cached data is considered "fresh" before TanStack Query will consider re-fetching it in the background.
- **`enabled`** — a boolean that tells TanStack Query whether it should run this query _at all_. When `false`, the query simply doesn't fire. This is exactly what you want when the query depends on some piece of state (like `focusedOrder`) that might not exist yet.

**Why `!!focusedOrder` instead of just `focusedOrder`?**

- A single `!` performs logical negation _and_ implicitly coerces its operand to a boolean. `!focusedOrder` means "is this falsy?" and returns `true`/`false`.
- Applying `!` a second time (`!!focusedOrder`) negates that result back to the _original_ truthiness — but now it's a **guaranteed real boolean** (`true`/`false`) rather than possibly a truthy/falsy non-boolean value (like a string, number, or `undefined`).
- This is a very common JavaScript idiom for **explicit boolean coercion** — forcing a value into `true`/`false` rather than relying on implicit truthiness elsewhere in the code. It's not strictly required in every case (some APIs happily accept any truthy/falsy value), but it's widely used for clarity and to avoid subtle bugs when a strict boolean is expected.

### Rendering the Loaded Data

```jsx
{focusedOrder ? (
  <Modal>
    <h2>Order #{focusedOrder}</h2>
    {isLoadingPastOrder ? (
      <p>Loading...</p>
    ) : (
      <table>
        <thead>
          <tr>
            <td>Image</td>
            <td>Name</td>
            <td>Size</td>
            <td>Quantity</td>
            <td>Price</td>
            <td>Total</td>
          </tr>
        </thead>
        <tbody>
          {pastOrderData.orderItems.map((pizza) => (
            <tr key={`${pizza.pizzaTypeId}-${pizza.size}`}>
              <td><img src={pizza.image} alt={pizza.name} /></td>
              <td>{pizza.name}</td>
              <td>{pizza.size}</td>
              <td>{pizza.quantity}</td>
              <td>{priceConverter(pizza.price)}</td>
              <td>{priceConverter(pizza.total)}</td>
            </tr>
          ))}
        </tbody>
      </table>
    )}
    <button onClick={() => setFocusedOrder(null)}>Close</button>
  </Modal>
) : null}
```

**Key composition detail:** even if `focusedOrder` is truthy (so we know we _should_ show the modal), the actual order data may not have arrived from the network yet. That's a **second, independent condition** (`isLoadingPastOrder`) nested inside the first. Don't conflate "should this UI exist" with "is the data for this UI ready" — they are two different booleans that often need to be checked separately.

### Choosing Robust `key` Values

```jsx
key={`${pizza.pizzaTypeId}-${pizza.size}`}
```

The `key` prop must uniquely identify each item in a list _among its siblings_ so React can correctly track, reorder, and update items across renders without wiping out and rebuilding unrelated DOM nodes. A composite key (type + size, rather than just an array index or just the type ID) protects you against subtly wrong behavior if, say, a user orders the same pizza type in two different sizes — those need to remain distinct rows rather than being collapsed into one. Building resilient keys early is inexpensive and prevents painful refactors later if the list gains sorting/filtering/reordering features.

---

## 3. Custom Hooks vs. Plain Utility Functions

A subtle but important architectural distinction: **not every reusable piece of logic should be a hook.**

### 3.1 When Something _Should_ Be a Hook

```jsx
// useCurrency.js
import { useMemo } from "react";

export default function useCurrency(price) {
  const intl = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  });
  return intl.format(price);
}
```

This works fine as a hook **when called at the top level of a component**:

```jsx
function OrderRow({ price }) {
  const formattedPrice = useCurrency(price);
  return <td>{formattedPrice}</td>;
}
```

### 3.2 The Rules of Hooks — Why You _Can't_ Call a Hook Inside `.map()`

React enforces (via linting and internally) that **hooks must be called in the same order on every render**, and specifically:

- Only call hooks at the **top level** of a function component or custom hook — never inside loops, conditions, or nested functions like the callback passed to `.map()`.
- This is because React tracks hook state internally using **call order**, not variable names. If the number/order of hook calls changes between renders (e.g., because a list you're mapping over grew or shrank), React loses track of which state belongs to which hook call, causing subtle and hard-to-diagnose bugs.

```jsx
// ❌ Invalid — calling a hook inside a callback passed to .map()
{items.map((item) => {
  const price = useCurrency(item.price); // Rules of Hooks violation
  return <td key={item.id}>{price}</td>;
})}
```

### 3.3 The Fix — Extract the Logic as a Plain Function

If the logic doesn't actually need any hook capabilities (state, effects, refs, context) — it's just a **pure calculation** — it doesn't need to be a hook at all:

```jsx
// priceConverter.js
export function priceConverter(price) {
  const intl = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  });
  return intl.format(price);
}
```

Now it can be freely called inside a `.map()`, because it's just a normal function, with none of the Rules-of-Hooks restrictions:

```jsx
{items.map((item) => (
  <td key={item.id}>{priceConverter(item.price)}</td>
))}
```

> **Rule of thumb:** If your reusable logic needs `useState`, `useEffect`, `useRef`, `useContext`, or any other hook internally, it must be its own custom hook (and must follow the Rules of Hooks in its consumers). If it's just a pure, synchronous transformation of input → output with no internal React state, make it a plain function instead — it's simpler and more flexible.

---

## 4. Data Mutations with TanStack Query — `useMutation`

### 4.1 Queries vs. Mutations

- A **query** (`useQuery`) represents _reading_ data. It's conceptually idempotent — calling it repeatedly with the same key doesn't change anything server-side, so TanStack Query is free to cache it, re-run it in the background, deduplicate it, etc.
- A **mutation** (`useMutation`) represents _changing_ something on the server — creating, updating, or deleting a resource. TanStack Query deliberately does **not** auto-run mutations the way it does queries; a mutation is only executed when you explicitly invoke it, because calling it repeatedly could have real, unwanted side effects (e.g., submitting a form twice).

TanStack Query's `useMutation` is a unified wrapper regardless of whether the underlying HTTP verb is `POST`, `PUT`, `PATCH`, or `DELETE` — semantically, "something is changing on the server," and the exact REST verb choice is an implementation detail that doesn't change how you use the hook.

### 4.2 Writing the API Call

```jsx
// postContact.js
export default async function postContact(name, email, message) {
  const response = await fetch("/api/contact", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ name, email, message }),
  });

  if (!response.ok) {
    throw new Error("Failed to submit contact form");
  }

  return response.json();
}
```

**Why `throw new Error(...)` instead of just returning an error object?** TanStack Query specifically watches for thrown errors (or rejected promises) from your `mutationFn`/`queryFn`. If you throw, TanStack Query automatically catches it, sets `isError` to `true` on the mutation/query result, and exposes the error object to you — giving you a consistent, built-in error-handling channel instead of having to build your own.

**Why `return response.json()` and not `return await response.json()`?** `response.json()` itself returns a Promise. Since `postContact` is an `async` function, it _always_ implicitly wraps its return value in a Promise anyway. Returning a Promise (rather than awaiting it first) triggers **promise chaining** — the outer promise automatically "adopts" the state of the inner one. Functionally, both approaches produce the same observable result for the caller; awaiting first is slightly more explicit, while returning the promise directly is more concise. Neither is wrong.

### 4.3 Using `useMutation` in a Component

```jsx
import { useMutation } from "@tanstack/react-query";
import postContact from "../api/postContact";

function ContactRoute() {
  const mutation = useMutation({
    mutationFn: (e) => {
      e.preventDefault();
      const formData = new FormData(e.target);
      return postContact(
        formData.get("name"),
        formData.get("email"),
        formData.get("message")
      );
    },
  });

  return (
    <div>
      <h2>Contact</h2>
      {mutation.isSuccess ? (
        <h3>Submitted!</h3>
      ) : (
        <form onSubmit={mutation.mutate}>
          <input name="name" placeholder="Name" />
          <input name="email" type="email" placeholder="Email" />
          <textarea name="message" placeholder="Message" />
          <button>Submit</button>
        </form>
      )}
    </div>
  );
}
```

**Why `onSubmit={mutation.mutate}` and not `onSubmit={() => mutationFn(...)}` directly?**

`mutation.mutate` is _not_ your raw `mutationFn` — it's a wrapper function provided by TanStack Query. Calling `mutate` triggers internal bookkeeping first (setting `isPending`/`isLoading` state, resetting previous error state, tracking retries, updating cache on success, etc.) and _then_ calls your `mutationFn` under the hood. This is why you call `mutation.mutate(event)`, not the function you originally passed in — `mutate` is the public entry point that wraps your function with all of that machinery.

- **`mutation.isSuccess`** — becomes `true` once the mutation resolves successfully; commonly used to conditionally swap the form for a "thanks!" message, or to trigger a redirect.
- **`mutation.isError`** / **`mutation.error`** — populated automatically if the `mutationFn` throws.
- **`mutation.isPending`** (formerly `isLoading` in older versions) — `true` while the mutation is in flight; useful for disabling the submit button or showing a spinner.

---

## 5. Uncontrolled Forms

### 5.1 Controlled vs. Uncontrolled — the Core Distinction

- A **controlled form** has React explicitly own the input's value via state: every keystroke updates a piece of state via `onChange`, and the `value` prop is fed back into the `<input>`. React is the single source of truth for what's in the field at all times.

```jsx
const [name, setName] = useState("");
<input value={name} onChange={(e) => setName(e.target.value)} />
```

- An **uncontrolled form** lets the **browser's native DOM** manage the input's live value. React doesn't track keystrokes at all — you only step in **once**, at submit time, to read whatever the browser currently has in the form.

### 5.2 Why Uncontrolled Is Often the Better Default

Controlled forms feel like the "React way," but they add unnecessary overhead — a state variable, a change handler, and a re-render on every keystroke — for data you only actually need **once, at submission time**. Unless you need to _react_ to input changes as they happen, an uncontrolled form is simpler and cheaper.

**When you genuinely need a controlled form:** any time the UI has to reactively respond to keystrokes or selections _before_ submission — live validation feedback, character counters, or the classic **"double dropdown" problem**, where selecting a value in one `<select>` (e.g., choosing "cat" as an animal type) must immediately repopulate the options in a second, dependent `<select>` (e.g., cat breeds). That dependency between two live pieces of UI state can only be modeled by having React track the first input's value in state.

**When uncontrolled is the right call:** you're just gathering a batch of fields and shipping them off somewhere on submit (contact forms, simple create/search forms) — nothing on the page needs to react to the values as the user types.

### 5.3 Implementing an Uncontrolled Form with the Browser's `FormData` API

```jsx
function handleSubmit(e) {
  e.preventDefault();
  const formData = new FormData(e.target);
  const name = formData.get("name");
  const email = formData.get("email");
  const message = formData.get("message");
  // ... send to your API
}
```

- **`e.preventDefault()`** — stops the browser's default form submission behavior, which would otherwise trigger a full page navigation/reload (a `GET`/`POST` to the current URL). You want to intercept the submit event and handle it in JavaScript instead.
- **`e.target`** — for a `submit` event fired from a `<form>`, `e.target` **is the form element itself**.
- **`new FormData(formElement)`** — a native **browser API** (not part of React at all) that automatically walks the form and collects the current value of every named input, textarea, select, etc. inside it into a single `FormData` object.
- **`formData.get("fieldName")`** — retrieves the value of the field whose `name` attribute matches `"fieldName"`. This is why every input in the form must have a `name` attribute — that's the key `FormData` uses to look values up.

Because there's no `value`/`onChange` wiring anywhere, this whole form requires **zero component state** — React never re-renders as the user types; the browser handles all of that natively, and React only steps in at the moment of submission to harvest the final values.

---

## 6. Class Components & Error Boundaries

### 6.1 Why Class Components Still Matter

Function components + hooks have been the dominant, recommended way to write React for years, to the point where many developers never write a class component. However, class components are **not deprecated** — and there is (as of this writing) exactly one situation where a class component is _required_ rather than optional: **Error Boundaries**. There is currently no hook-based equivalent for catching rendering errors in a subtree the way class components can.

In practice, most teams don't hand-write their own error boundary class — they use the well-established `react-error-boundary` package (maintained by a former member of the React core team), which wraps this pattern in a ready-to-use component. Still, understanding how to write one yourself matters for two reasons: you'll encounter class components in any codebase old enough to predate widespread hooks adoption, and understanding the mechanics deepens your grasp of React's rendering/error model overall.

### 6.2 The Problem Error Boundaries Solve

By default, an **uncaught JavaScript error thrown during rendering** (e.g., from a failed API call whose result you try to read a property from) will bubble all the way up and **crash the entire application** — the user sees a blank page or a broken app shell instead of a graceful fallback.

An **Error Boundary** is a component that can catch errors thrown by its descendants during rendering, log them, and render a fallback UI instead of letting the crash propagate further up the tree.

### 6.3 Anatomy of a Class Component

```jsx
import { Component } from "react";

class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error("ErrorBoundary caught an error", error, info);
    // In production, send this to an error-tracking service
    // (e.g., Sentry, TrackJS) instead of just logging it.
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Uh oh!</h2>
          <p>There was an error with this page.</p>
          <Link to="/">Click here to go back to the home page</Link>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

Let's break down every piece of this:

- **`class ErrorBoundary extends Component`** — this is standard, unmodified ES2015/ES6 JavaScript class syntax. `Component` is a base class exported by React that provides the plumbing (lifecycle hooks, `this.state`, `this.setState`, `this.props`) that turns a plain class into a React component.
- **`state = { hasError: false }`** — class components hold **all** of their state as a **single object** (`this.state`), rather than as many independent `useState` calls. Every stateful class component starts by declaring its initial shape here.
- **`static getDerivedStateFromError(error)`** — a **static method**, meaning it's called directly on the class itself (`ErrorBoundary.getDerivedStateFromError(...)`) rather than on an _instance_ of the class (contrast with an instance method, which you'd only be able to call after doing something like `const eb = new ErrorBoundary()`). React calls this static method automatically the moment a descendant throws during rendering, specifically so the component can compute what its **new state** should be in response to that error — hence "derived state from error." Whatever object you return here gets merged into `this.state`.
- **`componentDidCatch(error, info)`** — called after the error has been caught, primarily intended as the place to perform **side effects** in response to the error — most commonly, logging it to an error-tracking service (Sentry, TrackJS, etc.), or just `console.error` during development.
- **`render()`** — every class component **must** define a `render()` method; it is functionally equivalent to the entire body of a function component — it returns the JSX to display. Here, the logic is: _if we're in an error state, show a fallback UI; otherwise, act as a transparent pass-through and render whatever was passed in as children._
- **`this.props.children`** — the class-component equivalent of destructuring `{ children }` in a function component. When there's no error, the boundary renders `this.props.children` unmodified — it's meant to be **invisible** when nothing has gone wrong, simply forwarding whatever was nested inside it.

### 6.4 Rule #1 of Class Components: No Hooks

Class components **cannot** use `useState`, `useEffect`, or any other hook — hooks are exclusively a function-component construct. If you need to use a custom hook's logic inside a tree that includes a class component, the common workaround is to call the hook in a small wrapping **function component**, and pass the result down as a **prop**:

```jsx
function ErrorBoundaryWithHooks(props) {
  const potd = usePizzaOfTheDay();
  return <ErrorBoundary potd={potd} {...props} />;
}
```

The class component itself never calls a hook directly — it just receives the hook's _output_ as an ordinary prop.

### 6.5 Legacy Lifecycle Methods (Pre-Hooks Equivalents)

Before `useEffect` existed, class components exposed specific, named lifecycle methods that map roughly onto different effect behaviors:

|Class lifecycle method|Function-component equivalent|
|---|---|
|`componentDidMount()`|`useEffect(() => { ... }, [])` — runs once, after the first render|
|`componentDidUpdate(prevProps, prevState)`|`useEffect(() => { ... }, [dep1, dep2])` — runs after updates when dependencies change|
|`componentWillUnmount()`|the **cleanup function** returned from `useEffect` — runs once when the component is removed|
|`constructor(props)`|none needed — function components don't require an explicit constructor step|

### 6.6 The `constructor` and `super(props)`

If a class component needs to do setup work at the moment of creation, it defines a `constructor`:

```jsx
class MyComponent extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
}
```

`super(props)` calls the constructor of the parent class (`Component`), passing `props` up to it — this is required so that React's internal `Component` base class can properly wire up `this.props` before your own constructor logic runs.

### 6.7 The Classic `this`-Binding Problem

This is one of the most notorious gotchas of class components, and a common JavaScript interview topic in its own right.

```jsx
class Example extends Component {
  state = { celebration: false };

  celebrateError() {
    this.setState({ celebration: true });
  }

  render() {
    return <button onClick={this.celebrateError}>Click</button>;
  }
}
```

**This will not work correctly as written**, because of how JavaScript's `this` binding works for regular (non-arrow) functions: **the value of `this` inside a regular function is determined by _how_ the function is called, not where it was defined.** When React invokes `this.celebrateError` later as an event handler callback, it's no longer being called as `someInstance.celebrateError()` — it's just invoked as a bare function reference — so `this` inside it ends up `undefined` (in strict mode / ES modules) rather than pointing back at the component instance. That means `this.setState` inside it will throw, because `this` isn't the component instance anymore.

**Two common fixes:**

**Option A — explicitly bind the method (commonly done in the constructor):**

```jsx
constructor(props) {
  super(props);
  this.celebrateError = this.celebrateError.bind(this);
}
```

**Option B — define the method as an arrow function (class field syntax):**

```jsx
celebrateError = () => {
  this.setState({ celebration: true });
};
```

**Why does the arrow function fix it?** Arrow functions do not have their own `this` binding at all — they capture `this` **lexically**, meaning from the surrounding scope _where they were defined_ (inside the class body, so `this` refers to the component instance), rather than from _where/how they're called_. Regular functions determine `this` dynamically based on their call-site; arrow functions determine it statically based on where they were written. This distinction — **lexical vs. dynamic `this` binding** — is precisely why arrow-function class properties became the idiomatic way to avoid manual `.bind()` calls everywhere.

### 6.8 Placement Rules for Error Boundaries

An error boundary can only catch errors thrown by components **rendered inside it** — it cannot catch an error thrown by itself or by a sibling/ancestor. This means you must wrap the component that might throw **from the outside**, in a separate parent component:

```jsx
function ErrorBoundaryWrappedPastOrderRoute() {
  return (
    <ErrorBoundary>
      <PastOrderRoute />
    </ErrorBoundary>
  );
}

export default ErrorBoundaryWrappedPastOrderRoute;
```

This is conceptually identical to a `try { ... } catch { ... }` block in plain JavaScript — the code that might throw has to be _inside_ the `try`, and the same logic applies here: `PastOrderRoute` has to be inside (a child of) `ErrorBoundary`, not the other way around, and not the same component.

**A common, effective strategy:** place a single error boundary near the very **root** of your app, wrapping everything, so that any uncaught rendering error anywhere in the tree — including unexpected 404s or routing edge cases — gets caught by one top-level fallback UI instead of crashing the whole page.

### 6.9 A Note on the Spread Operator (`{...props}`) — Use It Sparingly

```jsx
function PassthroughWrapper(props) {
  return <ErrorBoundary {...props} />;
}
```

The spread operator forwards _all_ of a component's props on to another component in one concise line, rather than naming each one individually (`prop1={props.prop1} prop2={props.prop2}`).

**Why not use this everywhere, given how concise it is?** Because it sacrifices **explicitness** — one of React's core strengths is that data flow is usually very traceable: you can look at a component and see exactly which props it receives and uses. Spreading props hides that: a reader can no longer tell, just by looking at the component, what data is actually flowing through it or where it's going.

**The one case where spreading is a reasonable trade-off:** a component that is explicitly meant to be a **transparent pass-through** — like the small wrapper functions shown above, whose entire purpose is to connect two other components together without knowing or caring about the specific shape of the data flowing through them. In that narrow case, spreading communicates its own kind of intent: _"this component intentionally doesn't know or care what's inside props — it's just a pipe."_ Outside of that specific scenario, prefer being explicit about exactly which props a component consumes.

---

## Common Pitfalls & Best Practices

- **Creating a new DOM node on every render inside a portal component.** Always guard element creation with `useRef` + a truthy check (`if (!elRef.current) { ... }`), never create it unconditionally in the render body.
- **Forgetting the cleanup function in `useEffect`.** Any effect that creates something external (DOM nodes, subscriptions, timers, event listeners) needs a matching `return () => { /* undo it */ }` — otherwise you get memory leaks that compound with every mount/unmount cycle.
- **Calling hooks inside loops, conditionals, or `.map()` callbacks.** This violates the Rules of Hooks and leads to React losing track of hook state between renders. If the logic doesn't need actual hook capabilities, extract it as a plain function instead.
- **Firing a data fetch based on state that might not exist yet.** Use TanStack Query's `enabled` option (or an equivalent guard) to prevent queries from running with `undefined`/`null` inputs.
- **Confusing "should this render" with "is the data ready."** These are two separate conditions and usually need two separate, nested checks (e.g., `focusedOrder` existing vs. `isLoadingPastOrder` being `false`).
- **Using weak or non-unique `key` values in lists.** Prefer a composite key derived from genuinely unique identifying fields, especially if the list might later support reordering, filtering, or duplicate-looking entries.
- **Reaching for a controlled form when you don't need one.** If nothing on the page needs to react to keystrokes before submission, an uncontrolled form using `FormData` is simpler and avoids unnecessary re-renders.
- **Forgetting `e.preventDefault()` on form submission**, which causes an unwanted full-page reload/navigation instead of a JavaScript-handled submit.
- **Trying to use hooks inside a class component.** This is simply not possible — hooks are exclusive to function components. Compose around it with a small function-component wrapper that calls the hook and passes the result down as a prop.
- **Placing an Error Boundary as a sibling of, or inside, the component it's supposed to protect.** An error boundary can only catch errors from its own **descendants** — it must wrap the risky component from a level above.
- **The classic `this` context bug in class component event handlers.** Regular (non-arrow) methods lose their `this` binding when passed as callbacks. Fix with `.bind(this)` in the constructor, or define the method as an arrow-function class field.
- **Overusing the spread operator (`{...props}`) for "convenience."** It hides what data is actually flowing through a component. Reserve it for components that are intentionally meant to be pure pass-throughs.

---

## Summary of Key Takeaways

- **Portals** (`createPortal`) let a component's logical position in the React tree stay put while its actual DOM output is rendered elsewhere — solving z-index/overflow problems for modals, tooltips, and similar overlay UI.
- **`useRef`** returns a mutable object that's stable across re-renders and does **not** trigger a re-render when mutated — ideal for holding DOM nodes or other values that must persist without being "reactive" state.
- **Always clean up side effects.** A function returned from `useEffect` runs on unmount (or before the effect re-runs) and is the correct place to undo whatever the effect set up.
- **`children`** is a reserved prop representing whatever JSX is nested between a component's tags — essential for any component meant to wrap arbitrary content.
- **Not all reusable logic needs to be a hook.** If it doesn't use `useState`/`useEffect`/etc. internally, make it a plain function so it can be called anywhere, including inside loops.
- **Queries read, mutations write.** TanStack Query's `useQuery` runs automatically (and can be cache-aware via `queryKey`/`staleTime`/`enabled`); `useMutation` only runs when you explicitly call `.mutate(...)`.
- **Uncontrolled forms** let the browser own input state natively; you only read values once, via the native `FormData` API, at submit time — simpler and cheaper than controlled forms unless you need live reactivity between fields.
- **Class components are still required for Error Boundaries** (as of this writing) — no function-component/hook equivalent exists yet for catching rendering errors in a subtree.
- **Class components can never use hooks.** Combine hooks with class components by calling the hook in a wrapping function component and passing the result down as a prop.
- **`this` binding is dynamic in regular functions and lexical in arrow functions** — this is the root cause of the classic "my class method doesn't work as an event handler" bug, fixable via `.bind(this)` or arrow-function class fields.
- **An error boundary can only catch errors from its descendants** — it must wrap the risky component from outside, never be a sibling to it or contain no children of its own.