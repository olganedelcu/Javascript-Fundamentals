# 🧠 JavaScript Primitives

## 👋 introduction

a **primitive** is a value that’s **not an object**.  
it’s the simple stuff: you can store it, pass it around, compare it, or print it.

primitives are **immutable** — you don’t change their internal value, you just create a new one.

---

## 🔢 the 7 primitive data types

---

### 1. **string**
used for text: usernames, messages, urls, etc.

```js
const raw = "  olga.2002  ";
const username = raw.trim().toLowerCase(); 
// removes extra spaces & converts to lowercase

console.log("5" + 1); // '51' (string + number = string)
console.log("5" - 1); // 4 (js converts string to number)
```

### 2. **number**

integers and decimals. used for prices, lists number, percentanges, maths operations...

```js
const grades = [9, 10, 5.9]
const sum = grades.reduce((sum, grades) => {sum + grades, 0})
```

### 3. **boolean**
just true or false. 
used for permissions, toggles, flags, "is this valid?"

```js
const email = "olganedelcuam@gmail.com";
const hasAt = email.include("@");
const hasDot = email.slipt("@")[1].include(".");

const isValidEmail = hasAt && hasDot;
if (!isValidEmail) console.log("It is not a valid email")
```

Truthiness(super practical):
```js
const input = ""
if(!input) = console.log("disable submit button");
```

### 4. **undefined**
means "not assigned" or "does not exit here".

```js
const config = { theme: "dark"};
const lang = config.language; // undefined
console.log(lang ?? "en") // fallback to "en"
```

### 5. **null**
"intentionally empty" - you are saying "this is empty on purpose".

```js
let selectedUserId = 42;

// later user clear selection 
selectedUserId = null;

if (selectedUserID === null) console.log("No user selected");
```

### 6. **bigint**
used for really big numbers (like massive IDs, ledgers).

```js
const id = 12234678765432567654n; 
const next = id + 1n; 

console.log(next); // outcome: 12234678765432567655n
```

### 7. **symbol**
used for unique identifiers - often in libraries to avoid name collisions, used for metadata - avoiding clashes with other code.

```js
const internalId = Symbol("internalId"); 

const user = { name: "Ana" };
user[internalId] = "x9a-23";

console.log(user.internalId); // undefined
console.log(user[internalId]); // "x9a-23"
```

### 8. **primitive vs object**
primitives behave like values 

```js
let a = "hello";
let b = a;
b = "bye";

console.log(a) // "hello"
console.log(b) // "bye"
```
🔸objects behave like references:
```js
let a = { msg: "hello" };
let b = a;
b.msg = "bye";

console.log(a.msg) // "bye"

```
## type coercion
cheat sheet for checking types:

```js
typeof "x" // "string" 
typeof 10 // "number"
typeof true // "boolean" 
typeof undefined // "undefined" 
typeof 10n // "bigint"  
typeof Symbol() // "symbol" 
typeof null // "object" 🤯 (yes, legacy js bug)
```

✅ up next: [async-promises.md](async-promises.md) → promises, async/await, and all that jazz