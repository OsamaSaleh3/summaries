
# Execution Context
## 1. Core Concept (The "What")

This is the foundational walkthrough of JavaScript execution basics: how the **Thread of Execution** moves line-by-line through code, and how **Memory** stores data, using simple function calls and execution contexts as the vehicle.

## 2. The Code

```javascript
const num = 3;

function multiplyBy2(inputNumber) {
  const result = inputNumber * 2;
  return result;
}

const output = multiplyBy2(num);
const newOutput = multiplyBy2(10);
```

## 3. Under the Hood (The "How")

- **Global Memory (Global Execution Context created)**
    
    - `num` → `3` stored
    - `multiplyBy2` → entire function definition saved (drawn as `f`)
    - `output` → uninitialized (not yet known — we see `()`, so we know a function call is coming)
- **Thread of Execution** hits `multiplyBy2(num)`:
    
    - `num` evaluates to `3` before being passed in
    - Parentheses `()` = trigger to **execute/invoke/call** the function
- **New Execution Context created** (Call Stack: this context sits on top of Global)
    
    - **Local Memory** created just for this run:
        - Parameter `inputNumber` = argument value `3`
        - `result` = `inputNumber * 2` → `6`
    - `return` → sends the **value** of `result` (not the label `result`) back out
    - Execution context **closes/pops off** the Call Stack — local memory is discarded
    - Returned value `6` assigned to global label `output`
- **Thread of Execution** hits `multiplyBy2(10)`:
    
    - New Execution Context created again (pushed onto Call Stack)
    - New **Local Memory**: `inputNumber = 10`, `result = 20`
    - `return` sends `20` out, context closes/pops off stack
    - `20` assigned to global label `newOutput`
- No Web APIs / Task Queue / Event Loop involved here — this is 100% **synchronous**, single-threaded execution.
    

## 4. Interview TL;DR (The "Why")

- **Function calls = mini-programs**: every invocation spins up a brand-new Execution Context with its own Local Memory (Variable Environment) and its own Thread running through it — this is the conceptual seed for understanding **Closures** and the **Call Stack** later.
- **Return exits and destroys**: when a function returns, only the **_value_** survives — the execution context (and all its local variables/labels) is popped off the stack and forgotten. Interviewers probe this to check you understand that variables aren't "leaked" between calls unless closures capture them.

---




# The Call Stack

## 1. Core Concept (The "What")

This segment explains how JavaScript uses the **Call Stack** to keep track of which function is currently running and where to return to once execution finishes. It also touches briefly on how functions are stored in memory versus other data types.

## 2. The Code

```javascript
function multiplyBy2(inputNumber) {
  const result = inputNumber * 2;
  return result;
}

const output1 = multiplyBy2(3);
const output2 = multiplyBy2(10);
```

## 3. Under the Hood (The "How")

**Memory (Variable Environment)**

- `multiplyBy2` function definition is saved once in the **Global Memory**.
- Functions can be saved directly or as references/pointers — this distinction matters later in **type coercion**, but has no consequence here.
- Defined once, but can be **invoked/reused as many times as needed**.

**Thread of Execution**

- Global code runs top-to-bottom, executing line by line.
- When `multiplyBy2(3)` is invoked, JS creates a **new Local Execution Context** — a fresh pair of _Thread of Execution_ + _Local Memory_, scoped just to that function call.
- Inside, `inputNumber` is bound to `3`, `result` is computed and stored locally.
- On hitting the `return` keyword, JS knows to **exit** that function's execution context.

**Call Stack**

- **Global Execution Context** is created first, automatically, the moment the file runs (like an implicit `main()`).
- When `multiplyBy2(3)` is called → **push** `multiplyBy2(3)` onto the Call Stack.
- Stack only cares about what's on **top** — that's the actively running code.
- On `return` → **pop** `multiplyBy2(3)` off the stack.
- Whatever is now on top (**Global**) becomes the active execution context again.
- Same sequence repeats for `multiplyBy2(10)`: push → run → return → pop → back to Global.
- The Global Execution Context is only removed when there's nothing left to run (no pending callbacks/code).

## 4. Interview TL;DR (The "Why")

- **The Call Stack is LIFO (Last In, First Out)** — it only ever tracks the _current_ execution context on top; this is why deeply nested/recursive calls can cause a **"Stack Overflow"**.
- Understanding the Call Stack is foundational before tackling **asynchronous JS** (Web APIs, Task Queue, Microtask Queue, Event Loop) — interviewers often build on this to test if you know _why_ the stack must be empty before the Event Loop pushes callback code back in.