
## Set

```js
let myData = [1, 1, 1, 2, 3, "A"];
// let myUniqueData = new Set([1, 1, 1, 2, 3]);
// let myUniqueData = new Set(myData);
// let myUniqueData = new Set().add(1).add(1).add(1).add(2).add(3);
let myUniqueData = new Set();

myUniqueData.add(1).add(1).add(1); // Duplicates ignored → Set {1}
myUniqueData.add(2).add(3).add("A"); // Add 2, 3, "A"
```

👉 At this point:

```js
myUniqueData = Set { 1, 2, 3, "A" }
```

---

```js
console.log(`Is Set Has => A ${myUniqueData.has("a".toUpperCase())}`);
```

- `"a".toUpperCase()` becomes `"A"`.
    
- `myUniqueData.has("A")` → `true`.
    

So it prints:

```
Is Set Has => A true
```

---

```js
console.log(myData); 
// [1, 1, 1, 2, 3, "A"]
```

---

```js
console.log(myUniqueData); 
// Set { 1, 2, 3, 'A' }
```

---

```js
console.log(myUniqueData.size); 
// 3 (1, 2, "A")
```

---

```js
console.log(myData[0]);  // 1
console.log(myUniqueData[0]); // undefined (Set has no index-based access)
```

---

```js
// myUniqueData.delete(2);
console.log(myUniqueData.delete(2)); 
// true (2 was deleted)
```

---

```js
console.log(myUniqueData); 
// Set { 1, 3, 'A' }

console.log(myUniqueData.size); 
// 2
```

---

```js
myUniqueData.clear();
console.log(myUniqueData); 
// Set(0) {}

console.log(myUniqueData.size); 
// 0
```

---

✅ **Summary:**

- `Set` stores unique values only → duplicates are ignored.
    
- `.add(value)` inserts values (ignores duplicates).
    
- `.has(value)` checks for existence.
    
- `.delete(value)` removes an element (returns `true` if removed, `false` if not found).
    
- `.size` gives the number of unique elements.
    
- Unlike arrays, you **cannot access a `Set` item by index** (e.g., `myUniqueData[0]` is `undefined`).
    

---
