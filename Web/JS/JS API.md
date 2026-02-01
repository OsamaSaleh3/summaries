

---

# 🚀 JavaScript API & Async — Complete Beginner-Friendly Notes (Full)

---

## JSON Converting

```javascript
const myJsonObjectFromServer = '{"Username": "Osama", "Age": 39}';
console.log(typeof myJsonObjectFromServer); // string

const myJsObject = JSON.parse(myJsonObjectFromServer);
console.log(typeof myJsObject, myJsObject); // object { Username: "Osama", Age: 39 }

myJsObject.Username = "Elzero";
myJsObject.Age = 40;

const myJsonObjectToServer = JSON.stringify(myJsObject);
console.log(typeof myJsonObjectToServer, myJsonObjectToServer);
// string {"Username":"Elzero","Age":40}
```

`JSON.parse` converts JSON string → JS object.  
`JSON.stringify` converts JS object → JSON string.

---

## Call Stack and Event loop with callback Queue

---

## 🔹 Call Stack

- A stack (LIFO) where JavaScript keeps track of function calls.
    
- When a function is called → pushed on top.
    
- When it finishes → popped off.
    
- JS is **single-threaded**, so only one function runs at a time.
    

---

## 🔹 Web APIs

- Provided by the browser (like `setTimeout`, `fetch`, DOM events).
    
- When you call them, they run **outside** the call stack.
    
- Once finished, they hand back the callback to the **Callback Queue**.
    

---

## 🔹 Event Loop

- The **event loop** constantly checks:
    
    1. Is the call stack empty?
        
    2. If yes → take the first callback from the **queue** and push it onto the stack.
        
- This ensures non-blocking, asynchronous behavior (e.g., timers, promises, network requests).
    

---

### 🔧 Example:

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

console.log("3");
```

**Execution order:**

1. `"1"` is logged immediately (call stack).
    
2. `setTimeout` is sent to **Web API**, which waits 0 ms then pushes its callback to the **queue**.
    
3. `"3"` is logged next (still in the call stack).
    
4. Event loop sees stack empty → pushes `"2"` callback from queue → `"2"` is logged.
    

✅ Output:

```
1
3
2
```

---

## AJAX

---

```javascript
let myRequest = new XMLHttpRequest();
```

- Creates a new XHR object that can send/receive data from a server.
    
- This works asynchronously (non-blocking).
    

---

```javascript
myRequest.open("GET", "https://api.github.com/users/elzerowebschool/repos");
```

- `open(method, url)` prepares the request.
    
- `"GET"` → request type.
    
- URL → API endpoint we want data from.
    
- At this point, the request is **configured but not sent**.
    

---

```javascript
myRequest.send();
```

- Sends the request to the server.
    
- After this, the **request goes to the Web API** (browser handles it).
    

---

```javascript
myRequest.onreadystatechange = function () {
  if (this.readyState === 4 && this.status === 200) {
    console.log(this.responseText);
  }
};
```

- `onreadystatechange` → callback fired whenever `readyState` changes.
    
- `readyState` values:
    
    - `0`: Request not initialized
        
    - `1`: Connection established
        
    - `2`: Request sent
        
    - `3`: Processing request
        
    - `4`: Request finished and response is ready ✅
        
- `status`:
    
    - `200`: Success
        
    - `404`: Not Found
        
    - `500`: Server Error
        
- In this case:
    
    - When `readyState === 4` (done) **and** `status === 200` (OK), we log `responseText` (the API’s JSON response).
        

---

### 🔹 Flow with Web API + Event Loop

1. JS puts `myRequest.send()` into **Web API** (browser).
    
2. Browser handles network request **outside call stack**.
    
3. When response arrives, callback (`onreadystatechange`) is placed into the **callback queue**.
    
4. Event loop checks → if call stack is empty, callback moves into stack → executes.
    

---

## Callback Hell

---

```javascript
function makeItRed(e) {
  e.target.style.color = "red";
}
document.querySelector(".test").addEventListener("click", makeItRed);
```

👉 Example of a **callback** — function passed to run later when an event happens.

---

```javascript
setTimeout(() => {
  console.log("Download Photo From URL");
  setTimeout(() => {
    console.log("Resize Photo");
    setTimeout(() => {
      console.log("Add Logo To The Photo");
      setTimeout(() => {
        console.log("Show The Photo In Website");
      }, 1000);
    }, 1000);
  }, 1000);
}, 1000);
```

👉 **Callback Hell (Pyramid of Doom)** — too many nested callbacks, hard to read, debug, or handle errors properly.

---

⚠️ **Why it’s a problem?**

- Code becomes **nested and messy**.
    
- Very **hard to maintain** for larger tasks.
    
- Error handling is complicated.
    
- Leads to **inversion of control** (we rely on external functions to call ours at the right time).
    

---

```javascript
const myPromise = new Promise((resolve, reject) => {
  let connected = true;
  if (connected) {
    resolve("Connection Established");
  } else {
    reject("Connection Failed");
  }
});
```

👉 A **Promise**: starts `pending`, then becomes `fulfilled` (resolve) or `rejected` (reject).

---

```javascript
myPromise.then(
  (value) => console.log(`Success: ${value}`),
  (error) => console.log(`Error: ${error}`)
);
```

👉 `.then()` → attaches success and error callbacks.

---

```javascript
myPromise
  .then((value) => console.log(`Step 1: ${value}`))
  .then(() => console.log("Step 2: Do something else"))
  .catch((err) => console.log(`Caught: ${err}`));
```

👉 **Chaining** → handle results in sequence; `.catch()` for errors.

---

✅ **Summary:** Promises let you write async code without **callback hell**, making it cleaner, chainable, and easier to manage.

---

```javascript
const myPromise = new Promise((resolve, reject) => {
  let employees = [];
  if (employees.length === 4) {
    resolve(employees); // success → pass employees array
  } else {
    reject("Number Of Employees Is Not 4"); // fail → go to catch
  }
});

myPromise
  .then((res) => {
    res.length = 2;         // keep first 2 employees
    return res;             // pass to next then
  })
  .then((res) => {
    res.length = 1;         // keep only 1 employee
    return res;
  })
  .then((res) => {
    console.log(`The Chosen Employee Is ${res}`); // final result
  })
  .catch((err) => console.log(err))  // handles rejection if condition fails
  .finally(() => console.log("The Operation Is Done")); // runs always
```

👉 **Details:**

- `Promise` checks employees list. If not exactly 4 → goes straight to **catch**.
    
- Each `.then()` receives the previous resolved value, transforms it, and passes it on.
    
- `.catch()` runs if any rejection/error happens in the chain.
    
- `.finally()` always runs (success or error) → good for cleanup or final messages.
    

---

```javascript
const getData = (url, { timeout = 8000 } = {}) => new Promise((resolve, reject) => {
  const req = new XMLHttpRequest();
  req.responseType = "json";
  req.onload = () => (req.status >= 200 && req.status < 300) ? resolve(req.response) : reject(new Error(`HTTP ${req.status}`));
  req.onerror = () => reject(new Error("Network error"));
  req.ontimeout = () => reject(new Error("Request timeout"));
  req.open("GET", url);
  req.timeout = timeout;
  req.send();
});
```

Promise-wrapped XHR with JSON parsing, HTTP status guard, network/timeout errors, and configurable timeout.

---

```javascript
getData("https://api.github.com/users/elzerowebschool/repos")
  .then(d => d.slice(0, 10))
  .then(d => console.log(d[0]?.name))
  .catch(err => console.error(err.message))
  .finally(() => console.log("Done"));
```

Chains transform (limit to 10), logs first repo name, and uses catch/finally for error handling and cleanup.

---

## Fetch API

```javascript
/*
  Fetch API
  - Return A Representation Of the Entire HTTP Response
*/

fetch("https://api.github.com/users/elzerowschool/repos")  // 1️⃣ Make a network request
  .then((result) => {                                     // 2️⃣ Handle the raw response object
    console.log(result);                                   // Logs the full Response object (status, headers, etc.)
    let myData = result.json();                            // 3️⃣ Parse the response body into a JS object (returns a Promise)
    console.log(myData);                                   // Logs a Promise (not the actual data yet)
    return myData;                                         // 4️⃣ Return the Promise for the next .then()
  })
  .then((full) => {                                       // 5️⃣ Once .json() Promise resolves, we get the parsed data
    full.length = 10;                                      // Shrink the array to only 10 items
    return full;                                           // Pass it to the next .then()
  })
  .then((ten) => {
    console.log(ten[0].name);                             // 6️⃣ Log the `name` property of the first repo
  });
```

---

### 🔍 Step-by-Step Explanation

1. **`fetch("https://api.github.com/users/elzerowschool/repos")`**
    
    - This sends an HTTP `GET` request to the GitHub API, asking for the list of repositories owned by the user `elzerowschool`.
        
    - `fetch()` immediately returns a **Promise** that will eventually resolve with a **Response** object (not the actual data yet).
        
2. **First `.then((result) => { ... })`**
    
    - The parameter `result` is a `Response` object (not the JSON).
        
    - When you log `result`, you see metadata like `status`, `statusText`, `headers`, and a `body` stream.
        
3. **`result.json()`**
    
    - `.json()` is an asynchronous method of the `Response` object.
        
    - It reads the response body stream and parses it into a JavaScript object (here, it will be an **array of repository objects**).
        
    - Important: `result.json()` returns another **Promise**, not the actual data. That’s why in your `console.log(myData)` you see a pending Promise.
        
4. **`return myData`**
    
    - This returns the Promise to the next `.then`.
        
    - The second `.then` waits until this Promise resolves before executing.
        
5. **Second `.then((full) => { ... })`**
    
    - By now, `full` contains the parsed JSON data from the GitHub API.
        
    - `full` is an **array of repository objects**. You set `full.length = 10;` to only keep the first 10 items (a quick way to truncate an array).
        
6. **Third `.then((ten) => { ... })`**
    
    - Now `ten` is the truncated array (first 10 repos).
        
    - `console.log(ten[0].name)` prints the **name** property of the first repository in that list.
        

---

### ⚠️ Important Notes

- When you logged `console.log(myData)` inside the first `.then`, it showed a **Promise** because `.json()` itself is asynchronous. To see the actual data, always handle it in the _next_ `.then`.
    
- If the network request fails (bad URL, offline, etc.), `.catch()` will run with the error.
    
- `finally()` executes no matter what — good for cleanup, stopping spinners, etc.
    
- Unlike `XMLHttpRequest`, `fetch` does not reject the Promise on HTTP errors (like `404` or `500`). It only rejects on **network failures**. That’s why you often need to manually check `result.ok` before calling `.json()`.
    

---

👉 So your two code blocks are basically **equivalents**:

- The first (using `fetch`) is the modern, cleaner approach.
    
- The second (commented-out `XMLHttpRequest`) is the old way that requires more boilerplate.
    

---

✅ With the Fetch version, if you want to properly handle errors (like a 404), you could tweak it like this:

```javascript
fetch("https://api.github.com/users/elzerowschool/repos")
  .then((res) => {
    if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
    return res.json();
  })
  .then((data) => {
    console.log("First 10 repos:", data.slice(0, 10));
  })
  .catch((err) => console.error("Something went wrong:", err))
  .finally(() => console.log("The Operation Is Done"));
```

This way you see the actual data instead of just the raw `Response`.

---

## Promise (ALL/ALL Settled/Race)

```javascript
/*
  Promise
  - All
  - All Settled
  - Race
*/

// 1️⃣ First promise: resolves after 5 seconds
const myFirstPromise = new Promise((res, rej) => {
  setTimeout(() => {
    res("Iam The First Promise");  // ✅ resolve with a string
  }, 5000);
});

// 2️⃣ Second promise: rejects after 1 second
const mySecondPromise = new Promise((res, rej) => {
  setTimeout(() => {
    rej("Iam The Second Promise"); // ❌ reject with a string
  }, 1000);
});

// 3️⃣ Third promise: resolves after 2 seconds
const myThirdPromise = new Promise((res, rej) => {
  setTimeout(() => {
    res("Iam The Third Promise");  // ✅ resolve with a string
  }, 2000);
});
```

Now, let’s break down the three different static methods:

---

### 🔹 `Promise.all([...])`

```javascript
Promise.all([myFirstPromise, mySecondPromise, myThirdPromise]).then(
  (resolvedValues) => console.log(resolvedValues),
  (rejectedValue) => console.log(`Rejected: ${rejectedValue}`)
);
```

- **What it does:** Runs all promises in parallel and waits until **all succeed**.
    
- If **all** promises resolve → you get an array of results in the same order as the promises.
    
    - Example: `["Iam The First Promise", "Iam The Second Promise", "Iam The Third Promise"]`
        
- If **any one** promise rejects → the whole `Promise.all` immediately rejects with that error, ignoring the others.
    
    - In this case, since `mySecondPromise` rejects after 1s, the entire `Promise.all` rejects with `"Rejected: Iam The Second Promise"`, even though the others may eventually resolve.
        

---

### 🔹 `Promise.allSettled([...])`

```javascript
Promise.allSettled([myFirstPromise, mySecondPromise, myThirdPromise]).then(
  (results) => console.log(results),
  (rej) => console.log(`Rejected: ${rej}`)
);
```

- **What it does:** Runs all promises in parallel, **waits for all to finish**, and always resolves.
    
- Instead of failing fast, it gives you an array of result objects with **status** (`"fulfilled"` or `"rejected"`) and either `value` or `reason`.
    
- Example output:
    
    ```json
    [
      { "status": "fulfilled", "value": "Iam The First Promise" },
      { "status": "rejected", "reason": "Iam The Second Promise" },
      { "status": "fulfilled", "value": "Iam The Third Promise" }
    ]
    ```
    

This is useful when you want to know the outcome of **all** promises, not just the first one that rejects.

---

### 🔹 `Promise.race([...])`

```javascript
Promise.race([myFirstPromise, mySecondPromise, myThirdPromise]).then(
  (res) => console.log(res),
  (rej) => console.log(`Rejected: ${rej}`)
);
```

- **What it does:** Returns the result of the **first promise to settle** (resolve OR reject).
    
- In this case:
    
    - `mySecondPromise` rejects after **1 second**.
        
    - `myThirdPromise` resolves after **2 seconds**.
        
    - `myFirstPromise` resolves after **5 seconds**.
        
- Since the second one finishes first (at 1s) with a **reject**, the `race` rejects immediately with `"Rejected: Iam The Second Promise"`.
    
- If `mySecondPromise` had resolved instead of rejecting, `Promise.race` would resolve with its value.
    

---

### ⚡ Summary

- **`Promise.all`** → waits for everything; rejects fast if any fail.
    
- **`Promise.allSettled`** → waits for everything; always resolves with results + statuses.
    
- **`Promise.race`** → resolves/rejects as soon as the **first** promise finishes.
    

---

## Async

---

```javascript
/*
  Async
  - Adding `async` before a function makes it always return a Promise.
  - Inside an `async` function you can use `await` for cleaner async code.
*/

// 🔹 First way: Manually creating a Promise (commented out)
/*
function getData() {
  return new Promise((res, rej) => {
    let users = [];
    if (users.length > 0) {
      res("Users Found");           // Resolve if array is not empty
    } else {
      rej("No Users Found");         // Reject if array is empty
    }
  });
}

getData().then(
  (resolvedValue) => console.log(resolvedValue),        // Success handler
  (rejectedValue) => console.log("Rejected " + rejectedValue) // Error handler
);
*/

// 🔹 Second way: Returning a resolved or rejected Promise directly (commented out)
/*
function getData() {
  let users = ["Osama"];
  if (users.length > 0) {
    return Promise.resolve("Users Found");  // shortcut for returning a resolved promise
  } else {
    return Promise.reject("No Users Found"); // shortcut for rejected promise
  }
}

getData().then(
  (resolvedValue) => console.log(resolvedValue),
  (rejectedValue) => console.log("Rejected " + rejectedValue)
);
*/

// 🔹 Third way: Using async/await (cleaner style)
async function getData() {
  let users = [];
  if (users.length > 0) {
    return "Users Found";          // automatically wrapped in a resolved Promise
  } else {
    throw new Error("No Users Found"); // throwing inside async → rejected Promise
  }
}

console.log(getData()); // Immediately logs: Promise { <rejected> Error: No Users Found ... }

getData().then(
  (resolvedValue) => console.log(resolvedValue),            // runs if resolved
  (rejectedValue) => console.log("Rejected " + rejectedValue) // runs if rejected
);
```

---

### 🔍 Detailed Explanation

1. **Plain `Promise` constructor (first block)**
    
    - You explicitly create a new `Promise`, pass it an executor `(res, rej) => {...}`.
        
    - Call `res("...")` to fulfill, or `rej("...")` to reject.
        
    - Consumers use `.then(successHandler, errorHandler)` to respond.
        
2. **Using `Promise.resolve` and `Promise.reject` (second block)**
    
    - Instead of manually creating a `new Promise`, you can return an already resolved or rejected Promise.
        
    - This is shorter and more readable.
        
3. **Using `async/await` (third block)**
    
    - Adding the `async` keyword makes the function automatically return a Promise.
        
    - If you `return` a value → it becomes `Promise.resolve(value)`.
        
    - If you `throw` an error inside → it becomes `Promise.reject(error)`.
        
    - So:
        
        - `return "Users Found"` resolves the Promise.
            
        - `throw new Error("No Users Found")` rejects the Promise.
            
4. **`console.log(getData())`**
    
    - Since `async` functions always return a Promise, you’ll see something like:
        
        ```
        Promise { <rejected> Error: No Users Found }
        ```
        
5. **`getData().then(...).catch(...)`**
    
    - To actually use the result, you chain `.then()` for success and `.catch()` (or the second arg to `.then`) for failure.
        
    - In your code, `users` is an empty array, so the `throw` runs, and the **rejected branch** executes → prints:
        
        ```
        Rejected Error: No Users Found
        ```
        

---

✅ **Key takeaway:**

- `async` functions are just syntactic sugar for Promises.
    
- `return` inside async → resolves.
    
- `throw` inside async → rejects.
    
- Use `.then`/`.catch` or `await` to handle the result.
    

---

## await

```javascript
/*
  Await
  - `await` only works inside an `async` function.
  - It pauses the function execution until the Promise settles (resolves or rejects).
  - With `await`, you can write async code that looks synchronous.
*/

// Create a Promise that will settle after 3 seconds
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    // Uncomment one of these lines to test resolve vs reject:

    // resolve("Iam The Good Promise");    // ✅ Resolves successfully
    reject(Error("Iam The Bad Promise")); // ❌ Rejects with an Error object
  }, 3000);
});

async function readData() {
  console.log("Before Promise");

  // Option 1: Handle success with then/catch (commented out)
  // myPromise.then((resolvedValue) => console.log(resolvedValue));

  // Option 2: Await the promise directly (will throw if it rejects)
  // console.log(await myPromise);

  // Option 3: Await with `.catch()` to handle rejection inline
  console.log(await myPromise.catch((err) => err));

  console.log("After Promise");
}

readData();
```

---

### 🔍 What happens here?

1. **`myPromise` creation**
    
    - A new Promise is created. After **3 seconds**, it will either:
        
        - **Resolve** with `"Iam The Good Promise"` (if you uncomment that line), or
            
        - **Reject** with `Error("Iam The Bad Promise")` (as written now).
            
2. **Calling `readData()`**
    
    - Because `readData` is marked `async`, it returns a **Promise** immediately.
        
    - Inside, `console.log("Before Promise")` runs first.
        
    - Then `await myPromise.catch(...)` pauses execution until `myPromise` settles.
        
3. **Handling rejection**
    
    - Since we used `myPromise.catch(...)`, the error is “caught” and turned into a **resolved value** (the error object).
        
    - So `await ...` does not throw; instead it returns that error object.
        
    - `console.log(await myPromise.catch(...))` prints something like:
        
        ```
        Error: Iam The Bad Promise
        ```
        
4. **After Promise**
    
    - Once the awaited promise settles, execution continues and `"After Promise"` is logged.
        

---

### ⚡ Key details

- **`await` only works inside `async` functions**. Using it at the top level causes a syntax error (unless you’re in an ES module environment that supports top-level await).
    
- If you `await somePromise` and it **resolves**, you get the value directly.
    
- If it **rejects**, `await` throws an exception. You normally catch it with `try...catch`.
    
- Wrapping `myPromise` with `.catch(err => err)` ensures that even if it rejects, it resolves to the error object, letting you handle it inline.
    

---

## try & catch & finally

---

```javascript
/*
  Async & Await With Try, Catch, Finally
*/

// A promise that resolves successfully after 3 seconds
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Iam The Good Promise");   // after 3s, resolve with this string
  }, 3000);
});

// Example 1: Using async/await with try–catch–finally
/*
async function readData() {
  console.log("Before Promise");
  try {
    console.log(await myPromise);     // waits for promise, then logs "Iam The Good Promise"
  } catch (reason) {
    console.log(`Reason: ${reason}`); // runs only if the promise rejects
  } finally {
    console.log("After Promise");     // always runs
  }
}

readData();
*/

// Example 2: Using fetch with async/await, try–catch–finally
async function fetchData() {
  console.log("Before Fetch");
  try {
    // `fetch` returns a promise that resolves with a Response object
    let response = await fetch("https://api.github.com/users/elzerowebschool/repos");

    // To get the actual JSON, you must also await response.json() (it's async)
    let data = await response.json();

    console.log(data);  // Logs the parsed JSON (an array of repo objects)
  } catch (reason) {
    // If fetch fails (network error, invalid URL, etc.), execution jumps here
    console.log(`Reason: ${reason}`);
  } finally {
    // This block always runs, success or failure → good place for cleanup
    console.log("After Fetch");
  }
}

fetchData();
```

---

### 🔍 Detailed Explanation

1. **`myPromise` example (`readData`)**
    
    - Because the promise resolves after 3s, the `await myPromise` line will pause the function until the value `"Iam The Good Promise"` is ready.
        
    - The `try` block logs that resolved value.
        
    - If instead you had `reject(...)` inside the promise, the `catch` block would run.
        
    - The `finally` block executes in **both cases** (resolved or rejected).
        
    
    Output (with resolve):
    
    ```
    Before Promise
    Iam The Good Promise
    After Promise
    ```
    
2. **`fetchData` example**
    
    - `fetch()` is an async API call. Awaiting it gives you a `Response` object.
        
    - To extract JSON data from the response, you must call and `await response.json()`.
        
    - If the network request fails or JSON parsing throws an error, the `catch` block handles it.
        
    - The `finally` block runs regardless — even if an error happened or not.
        

---
