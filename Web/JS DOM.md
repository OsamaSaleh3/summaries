
---

# 🌐 DOM Cheatsheet (with Examples)

---

## 🔹 What is the DOM?

The **DOM (Document Object Model)** is a representation of your webpage as a **tree of objects**.  
Each HTML element becomes a **node** in this tree.

👉 With JavaScript, you can **select elements**, **change content**, **modify styles**, and **react to user events**.

---

## 🟢 1. Selecting Elements

```html
<div id="my-div">Hello</div>
<p class="my-span">First</p>
<p class="my-span">Second</p>
```

```javascript
document.getElementById("my-div");       // select by ID
document.getElementsByTagName("p");      // all <p>
document.getElementsByClassName("my-span"); // by class
document.querySelector(".my-span");      // first match
document.querySelectorAll(".my-span");   // all matches
```

---

## 🟢 2. Accessing Elements

```html
<title>My Page</title>
<body>
  <p>One</p>
  <p>Two</p>
  <form>
    <input name="one" value="hello">
  </form>
  <a href="https://google.com">Google</a>
</body>
```

```javascript
document.title;        // "My Page"
document.body;         // entire <body>
document.getElementsByTagName("p")[1]; // <p>Two</p>
document.forms[0].one.value;  // "hello"
document.links[0].href;       // "https://google.com"
```

---

## 🟢 3. Changing Content

```html
<p id="msg">Old text</p>
```

```javascript
let msg = document.getElementById("msg");

msg.innerHTML = "Hello <b>World</b>";   // renders bold
msg.textContent = "Hello <b>World</b>"; // shows raw text
```

---

## 🟢 4. Working with Attributes

```html
<a id="link" href="https://example.com">Example</a>
```

```javascript
let link = document.getElementById("link");

link.getAttribute("href");      // "https://example.com"
link.setAttribute("href", "https://twitter.com");
link.hasAttribute("href");      // true
link.removeAttribute("href");
```

---

## 🟢 5. Creating & Adding Elements

```javascript
let div = document.createElement("div");
div.className = "product";
div.textContent = "Product One";

document.body.appendChild(div);
```

👉 Creates a new `<div class="product">Product One</div>` at the end of `<body>`.

---

## 🟢 6. DOM Traversal

```html
<ul id="list">
  <li>Apple</li>
  <li>Banana</li>
</ul>
```

```javascript
let list = document.getElementById("list");

list.children;              // [<li>Apple</li>, <li>Banana</li>]
list.firstElementChild;     // <li>Apple</li>
list.lastElementChild;      // <li>Banana</li>
list.parentElement;         // <body>
```

---

## 🟢 7. Insert & Remove

```html
<ul id="fruits"><li>Apple</li></ul>
```

```javascript
let fruits = document.getElementById("fruits");

fruits.append("Banana");    // <ul><li>Apple</li>Banana</ul>
fruits.prepend("Orange");   // adds at start
fruits.firstChild.remove(); // removes "Orange"
```

---

## 🟢 8. Classes

```html
<div id="box" class="red"></div>
```

```javascript
let box = document.getElementById("box");

box.classList.add("big");      // add new class
box.classList.remove("red");   // remove class
box.classList.toggle("hidden"); // add/remove
box.classList.contains("big"); // true
```

---

## 🟢 9. Styling

```html
<p id="txt">Hello</p>
```

```javascript
let txt = document.getElementById("txt");

txt.style.color = "red";
txt.style.fontWeight = "bold";

txt.style.cssText = "color: green; font-size: 20px;";
txt.style.removeProperty("color");
txt.style.setProperty("font-size", "40px", "important");
```

---

## 🟢 10. Events (Mouse & Window)

```html
<button id="btn">Click Me</button>
```

```javascript
let btn = document.getElementById("btn");

btn.onclick = () => alert("Clicked!");
window.onload = () => console.log("Page loaded!");
window.onscroll = () => console.log("Scrolling...");
```

---

## 🟢 11. Forms & Validation

```html
<form id="form">
  <input id="username" />
  <button>Submit</button>
</form>
```

```javascript
let form = document.getElementById("form");
let username = document.getElementById("username");

form.onsubmit = e => {
  e.preventDefault();
  if (username.value !== "" && username.value.length <= 10) {
    console.log("Valid username!");
  }
};
```

---

## 🟢 12. Event Listeners & Delegation

```html
<button id="btn">Click</button>
<div class="clone">Box</div>
```

```javascript
let btn = document.getElementById("btn");
btn.addEventListener("click", () => console.log("Button clicked"));

// Delegation
document.addEventListener("click", e => {
  if (e.target.className === "clone") {
    console.log("I am Cloned");
  }
});
```

---

## 🟢 13. Cloning

```html
<p id="para">Hello</p>
<div id="container"></div>
```

```javascript
let para = document.getElementById("para");
let copy = para.cloneNode(true); // with text
copy.id = "new-para";

document.getElementById("container").appendChild(copy);
```

---
