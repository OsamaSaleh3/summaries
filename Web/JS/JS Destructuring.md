```javascript
let arr = ["osama", "khaled", "abdelmajid", "saleh"];
let [a, b, c, d] = arr;
console.log(a, b, c, d); // osama khaled abdelmajid saleh
```

Array destructuring → assigns each element to a variable.

---

```javascript
let [x, , y, z] = arr;
console.log(x, y, z); // osama abdelmajid saleh
```

Skipping items → commas skip elements during destructuring.

---
```javascript
let arr = ["osama","khaled","abdelmajid",["rami","mohammed",["sami","jamal"]]];
let [,,, [a,, [, b]]] = arr;

console.log(a); // rami
console.log(b); // jamal
```

Nested destructuring → extract values from deep inside arrays.

---

```javascript
let book = "video";
let video = "book";

[book, video] = [video, book];
console.log(book, video); // "book" "video"
```

Array destructuring swap → swaps the values of two variables.

---

```javascript
const user = { name: "osama", age: 21, title: "developer", country: "jordan" };
let { name, age, title, country } = user;

console.log(name, age, title, country); // osama 21 developer jordan
```

Object destructuring → extracts properties into variables.

---

```javascript
({ name, age, title, country } = user);
console.log(name, age, title, country); // osama 21 developer jordan
```

Re-destructuring → parentheses are needed when reassigning variables without `let` or `const`.

---

```javascript
const user = {
  theName: "Osama",
  theAge: 39,
  theTitle: "Developer",
  theCountry: "Egypt",
  theColor: "Black",
  skills: { html: 70, css: 80 },
};

const {
  theName: n,
  theAge: a,
  theCountry,
  theColor: co = "Red",
  skills: { html: h, css },
} = user;

console.log(n, a, theCountry, co); 
// Osama 39 Egypt Black
console.log(`My HTML Skill Progress Is ${h}`);
console.log(`My CSS Skill Progress Is ${css}`);
```

Object destructuring with renaming, defaults, and nested properties.

---

```javascript
const { html: skillOne, css: skillTwo } = user.skills;
console.log(`My HTML Skill Progress Is ${skillOne}`);
console.log(`My CSS Skill Progress Is ${skillTwo}`);
```

Destructuring nested object directly → extracts `html` and `css`.

---

```javascript
const user = {
  theName: "Osama",
  theAge: 39,
  skills: ["HTML", "CSS", "JavaScript"],
  addresses: { egypt: "Cairo", ksa: "Riyadh" },
};

let { theName: n, theAge: a, skills: [x, y, z], addresses: { egypt } } = user;

console.log(n, a, x, y, z, egypt); 
// Osama 39 HTML CSS JavaScript Cairo
```

Nested destructuring → extracts from object, array, and sub-object in one step.

---

