### Lesson 1: Promises, async, and the event loop

**1. Core Concept (The "What"):**
JavaScript operates synchronously by default, executing code line-by-line within a single thread of execution. This foundational model relies on a Global Execution Context and a Call Stack to track running functions.

**2. The Code:**

```javascript
const num = 3;
function multiplyBy2(inputNumber) {
  const result = inputNumber * 2;
  return result;
}
const output = multiplyBy2(num);
const newOutput = multiplyBy2(10);

```

**3. Under the Hood (The "How"):**

* **Thread of Execution:** JavaScript goes through the code line-by-line, doing only one thing at a time.


* **Memory (Variable Environment):** The engine stores the constant `num` (assigned `3`) and the function definition `multiplyBy2` into the global memory.


* **Call Stack:** When `multiplyBy2(num)` is invoked, it is pushed onto the top of the Call Stack.


* **Local Memory:** Inside the execution context of `multiplyBy2`, the parameter `inputNumber` is assigned the argument `3`, and `result` evaluates to `6` before returning to global memory.


* **Call Stack:** Once the `return` statement is hit, the execution context is popped off the Call Stack, and the thread returns to the global context to evaluate the next line.



**4. Interview TL;DR (The "Why"):**

* **Single-Threaded Synchronous Nature:** JavaScript can only execute one command at a time directly; it never leaves a line unfinished before moving to the next. This predictable line-by-line behavior is the baseline before asynchronous concepts are introduced.



---

### Lesson 2: Two-Pronged Background Work

**1. Core Concept (The "What"):**
Modern asynchronous JavaScript utilizes "two-pronged" functions like `fetch` to simultaneously create a placeholder object in JavaScript and initiate background work in the Web Browser APIs.

**2. The Code:**

```javascript
function display(data) { /* ... */ }
function printHello() { /* ... */ }
function blockFor300ms() { /* ... */ }

setTimeout(printHello, 0);
const futureData = fetch('https://tiktok.com/will');
futureData.then(display);

```

**3. Under the Hood (The "How"):**

* **Web Browser APIs:** `setTimeout` sets up a timer in the web browser, immediately completing (at 0ms) and sending `printHello` to the Callback Queue.


* **Memory (Variable Environment):** The `fetch` call returns a Promise object stored in `futureData` with two hidden properties: `value` (initially undefined) and `onFulfilled` (an empty array).


* **Web Browser APIs:** Simultaneously, `fetch` triggers an automatic GET network request to TikTok's server.


* **Thread of Execution:** The `.then()` method grabs the `display` function definition and pushes it into the `onFulfilled` array on the `futureData` Promise object.



**4. Interview TL;DR (The "Why"):**

* **The Promise Architecture:** `fetch` does not freeze the Call Stack; it offloads work to Browser APIs while immediately returning a live object (Promise) to JavaScript memory, allowing the single thread to continue.



---

### Lesson 3: The Microtask Queue

**1. Core Concept (The "What"):**
When dealing with asynchronous operations, the Event Loop prioritizes Promise-deferred functions by placing them in a specialized queue that executes before standard callbacks.

**2. The Code:**

```javascript
setTimeout(printHello, 0);
const futureData = fetch('https://tiktok.com/will');
futureData.then(display);
blockFor300ms();
console.log('me first');

```

**3. Under the Hood (The "How"):**

* **Call Stack:** `blockFor300ms` runs on the Call Stack, blocking the main thread for 300 milliseconds.


* **Web Browser APIs:** The network request finishes at roughly 270 milliseconds, returning data ("cute puppy") to update the `futureData.value`.


* **Microtask Queue:** Because the Promise value updated, `display` is pushed into the Microtask Queue.


* **Event Loop:** After `console.log('me first')` finishes at 303 milliseconds, the global code is complete and the Call Stack is empty.


* **Thread of Execution:** The Event Loop strictly prioritizes the Microtask Queue over the Callback Queue, pushing `display` to the Call Stack before `printHello`.



**4. Interview TL;DR (The "Why"):**

* **Queue Priority:** Promise callbacks (Microtask Queue) always run before standard API callbacks (Callback Queue), which is a massive interview "gotcha" regarding execution order.



---

### Lesson 4: Promise Abort Control

**1. Core Concept (The "What"):**
While Promises allow for cleaner, pseudo-synchronous code, developers can also dictate early cancellations for background tasks (like `fetch`) using `AbortSignal` and timers.

**2. The Code:**

```javascript
const signal = AbortSignal.timeout(200);
fetch('https://tiktok.com/will', { signal })
  .then(display)
  .catch(handleError);

```

**3. Under the Hood (The "How"):**

* **Web Browser APIs:** `AbortSignal.timeout(200)` spins up a 200ms timer directly linked to the network request inside the browser.


* **Memory (Variable Environment):** It returns an object containing an `aborted` boolean and a `reason` property into JavaScript.


* **Thread of Execution:** If the 200ms timer completes before the server responds, it aborts the network task.


* **Call Stack / Microtask Queue:** The Promise rejects, bypassing the `.then()` array completely and pushing the function registered via `.catch()` (the reject reaction) into the Microtask Queue.



**4. Interview TL;DR (The "Why"):**

* **Race Conditions under the hood:** By connecting a Web Browser timer to a network request, developers avoid blocking the UI while gaining strict control over unresponsive servers, triggering error handling cleanly without hanging promises.



---

### Lesson 5: The Need for Facade Functions

**1. Core Concept (The "What"):**
Because JavaScript has only one Thread of Execution, running heavy tasks natively blocks all further code; therefore, JavaScript uses "facade functions" to interface with external Web Browser APIs.

**2. The Code:**

```javascript
function getVideos() { /* Heavy native operation */ }
function printHello() { console.log('hello'); }

setTimeout(printHello, 1000);
console.log('me first');

```

**3. Under the Hood (The "How"):**

* **Thread of Execution:** Natively blocking the thread with large loops halts the entire application.


* **Memory (Variable Environment):** JavaScript alone only contains the Call Stack, Memory, and Thread of Execution.


* **Web Browser APIs:** Features like the DOM, Local Storage, Timers, and Network Fetching are not actually JavaScript; they exist in the browser (or Node background).


* **Call Stack:** Built-in functions like `setTimeout` or `document` act as facades; they are called in JS but immediately pass work to the browser so the single thread can move on to `console.log('me first')`.



**4. Interview TL;DR (The "Why"):**

* **The JS Environment:** A common misconception is that features like `setTimeout` or the `console` are pure JavaScript; they are actually browser APIs accessed via JS labels to preserve non-blocking performance.



---

### Lesson 6: Web APIs and Asynchronous Delay

**1. Core Concept (The "What"):**
Tracing exact background API execution proves how JavaScript continues executing synchronous code linearly while offloading timers to the browser.

**2. The Code:**

```javascript
function printHello() {
  console.log('hello');
}
setTimeout(printHello, 1000);
console.log('me first');

```

**3. Under the Hood (The "How"):**

* **Memory (Variable Environment):** `printHello` is saved in global memory.


* **Web Browser APIs:** `setTimeout` acts as an interface, spinning up a timer for 1000ms in the browser and retaining a reference to `printHello`.


* **Thread of Execution:** Execution finishes on the `setTimeout` line instantly and moves to `console.log('me first')` at 1 millisecond.


* **Call Stack:** At 1000ms, the timer completes in the browser, and `printHello` asks to get back onto the Call Stack to run, opening a new Execution Context.



**4. Interview TL;DR (The "Why"):**

* **Non-Blocking Behavior:** Timeouts do not pause JavaScript's thread. The JS engine delegates the counting to the browser to ensure the page remains interactive while the timer ticks down.



---

### Lesson 7: The Callback Queue's Strictest Rule

**1. Core Concept (The "What"):**
Even if an asynchronous task completes instantly, its callback function is entirely barred from executing until all global synchronous code finishes.

**2. The Code:**

```javascript
function printHello() { console.log('hello'); }
function blockFor1Sec() { /* computationally taxing loop */ }

setTimeout(printHello, 0);
blockFor1Sec();
console.log('me first');

```

**3. Under the Hood (The "How"):**

* **Web Browser APIs:** `setTimeout` sets a 0ms timer that completes immediately in the background.


* **Callback Queue:** `printHello` is instantly pushed to the Callback Queue (Task Queue) and waits.


* **Call Stack:** `blockFor1Sec` goes on the Call Stack, completely blocking the thread for 1 full second (1000ms).


* **Thread of Execution:** After 1000ms, `console.log('me first')` runs. Only after the global context finishes completely (1002ms) is `printHello` finally added to the Call Stack.



**4. Interview TL;DR (The "Why"):**

* **Zero-Delay Myth:** A `setTimeout` of 0 does not mean the code runs instantly; it simply guarantees the callback gets placed in the Callback Queue instantly, subject strictly to the clearing of the global execution context.



---

### Lesson 8: The Event Loop Check

**1. Core Concept (The "What"):**
The mechanism regulating the flow of background functions back into the main thread is the Event Loop, operating under a strict two-part validation.

**2. The Code:**
*(Conceptual continuation of the asynchronous environment architecture).*

**3. Under the Hood (The "How"):**

* **Event Loop:** The Event Loop sits between the Callback Queue and the Call Stack, constantly checking on repeat.


* **Thread of Execution:** It asks two strict questions: Is the Call Stack entirely empty? AND Has all global code completely finished running?.


* **Call Stack:** If a blocking function or millions of subsequent lines of code are still executing, the Event Loop denies entry.


* **Callback Queue:** Once both rules are met, it dequeues the first function waiting in the Callback Queue and pushes it onto the Call Stack to be executed.



**4. Interview TL;DR (The "Why"):**

* **Predictability Standard:** The strict Event Loop rules prevent random background responses from interrupting running functions, guaranteeing predictable synchronous order regardless of network speeds.



---

### Lesson 9: Transition to Promises

**1. Core Concept (The "What"):**
ES5 callback patterns (like `setTimeout`) suffered from poor data tracking and "callback hell", leading to ES6 introducing Promises—a structure that returns a synchronous trackable object immediately.

**2. The Code:**

```javascript
const futureData = fetch('https://tiktok.com/will');

```

**3. Under the Hood (The "How"):**

* **Web Browser APIs:** `fetch` triggers a GET request in the browser's background network features.


* **Memory (Variable Environment):** Simultaneously, `fetch` returns a Promise object to the `futureData` variable.


* **Thread of Execution:** This Promise object has a hidden `result` (or `value`) property sitting as undefined, and a hidden `onFulfilled` array.


* **Call Stack:** Instead of just sending a callback blindly into the void, JS maintains a synchronous handle (the Promise) attached to the async work.



**4. Interview TL;DR (The "Why"):**

* **The Two-Pronged Facade:** Understanding `fetch` requires recognizing it initiates an async Web Browser API request while simultaneously generating a sync JavaScript Promise object, forming a bridge between the two environments.



---

### Lesson 10: `.then` and Data Hydration

**1. Core Concept (The "What"):**
The `.then` method allows developers to attach synchronous functions to the Promise object so they are auto-triggered when the background Web API retrieves data.

**2. The Code:**

```javascript
function display(data) { console.log(data); }
const futureData = fetch('https://tiktok.com/will');
futureData.then(display);
console.log('me first');

```

**3. Under the Hood (The "How"):**

* **Memory (Variable Environment):** `.then(display)` accesses the hidden `onFulfilled` array (Promise Fulfill Reactions) inside the `futureData` object and pushes the `display` function into it.


* **Thread of Execution:** JS continues synchronously, logging 'me first'.


* **Web Browser APIs:** Once the network request completes, it updates the `result` property of the `futureData` Promise object with the retrieved string ("cute puppy").


* **Call Stack:** This data update automatically pushes the `display` function into the Microtask Queue, subsequently placing it on the Call Stack where the `data` parameter is automatically hydrated with the result.



**4. Interview TL;DR (The "Why"):**

* **Inversion of Control:** Instead of manually waiting for data and invoking functions, we register the function on the Promise object, trusting the JavaScript engine to auto-invoke it with the correct argument once the data arrives.



---

### Lesson 11: Memory Persistence and Promise Resolution

**1. Core Concept (The "What"):**
Promises defined in the global memory (or local contexts) persist beyond standard garbage collection because the Web Browser maintains an active reference to them during background work.

**2. The Code:**
*(Conceptual wrap-up of the `fetch` and `.then` architecture).*

**3. Under the Hood (The "How"):**

* **Memory (Variable Environment):** Even if an object is defined locally, it sits on the memory heap.


* **Web Browser APIs:** The browser's active network request holds an implementation-level reference directly back to that specific Promise object in JavaScript memory.


* **Thread of Execution:** Because of this reference, when the background task resolves, the browser locates the object and updates its hidden `result` property.


* **Microtask Queue:** The update triggers the associated `onFulfilled` callbacks to jump onto the Microtask queue, executing on the Call Stack only when global contexts clear.



**4. Interview TL;DR (The "Why"):**

* **Cross-Environment Referencing:** Background APIs don't hold "copies" of the object; they hold memory references back to the JS runtime. This linkage prevents the Promise from disappearing and allows external APIs to mutate JavaScript state.