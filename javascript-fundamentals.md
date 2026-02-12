# ⏳ javascript + typescript fundamentals

## var vs let vs const
- var: old, function scoped, hoisted
- let: block-scoped, reassignable
- cosnt: block-scoped, not reassignabe
```js
var car = "car"; // function scoped
let a = 5; // block scoped
const b = 1; // cannot reassign 
```

## 🌀 scope, hoisting, closures
- scope = where variables live
- hoisting = var + functions get "lifted" to top
- closures = inner function "remember" outer variables

#### 🧙‍♀️ Hoisting in JavaScript
##### definition
hoising is when JS "remembers" all your variables & function declarations **before** it actually runs the code.

###### memory allocation phase: 
- all **var**, **function**, and **class** declarations are stored in memory. 
- JS does this automatically, **before** executing line-by-line.

###### key insights: 
- only declarations are hoisted -  not initializations
- **let** and **const** are hoisted too, but not accessible before declaration
- function declaration are hoisted fully, function expressions are not.

## 🧳 Closures
a **closure** is when an inner function remembers the variables from its outer function, even after the outer function has finished running.

they are not only in JS, but it means that functions **enclose** different variables that are there when the function is **declared**. And by **enclosing**, it means they **remember** them. So basically it means whenever you import or use a function, you **also get by default** all the **scope** where that function was created.

so if there is a variable at the root level, the function has access to that forever, until you never use that function again.
```js
const name = "Olga";

export function sayHello() {
  console.log(name);
}
```

iIf someone imports and runs this, the function will still work because even if they provide an argument or not, it will actually look in the function scope and then in the global and find this variable.

Even if this variable is not present where this function is being called, as long as it’s present in the module — (name = "Olga" still shows).

Another way you could show this is if you return another function. Let’s say you actually create:
```js
export function createSayHello() {
  const name = "Olga"; // now 'name' is in the scope of createSayHello

  return function sayHello() {
    console.log(name);
  }
}
```
then
```js
export function createSayHello() {
  const name = "Olga"; // now 'name' is in the scope of createSayHello

  return function sayHello() {
    console.log(name);
  }
}
```
then:
```js
const sayHelloOlga = createSayHello();
sayHelloOlga(); // still works ✅
```
so scope is basically whatever you see that is surrounded by curly brackets {}. So if I use sayHello outside here, it will still work.

Let’s rewrite the full example again just to make it clearer:
```js
export function createSayHello() {
  // 'name' was enclosed by sayHello
  const name = "Olga";

  return function sayHello() {
    console.log(name);
  };
}

const sayHelloOlga = createSayHello();
sayHelloOlga(); // still works
```
i would get no **exception**, even if in the **scope I do not have a name**. So the function is using a variable called name. it is not present in this global scope. however, because the function was created in this scope, it has access to this variable.

so that is **closure** — because we can say that name was enclosed by sayHello.
you do not **only get the function — you get the whole scope**, the whole lexical environment 🌍

- used for data privacy, stateful functions, and async logic.

### ⚠️ what is the drawback of this?
it puts a lot of **pressure on memory** 🧠 because everything that is in the scope is still referenced by this function as long as the function is being used.

so we cannot **remove** it from **memory**, we cannot release that memory by the **garbage collection** algorithm.

so basically, **JavaScript cannot clean it up** — it cannot get rid of anything that is in the **closure** scope of this function. and that means you put a lot of pressure on memory.

that is why traditionally we all know that JavaScript is not a super memory-efficient language — it actually needs a lot of **RAM memory** 💾.

## ➡️ arrow functions

arrow functions are a shorter syntax for writing functions, introduced in ES6.
```js
// traditional function
function add(a, b) {
  return a + b;
}

// arrow function
const add = (a, b) => a + b;

// with a body (needs explicit return)
const multiply = (a, b) => {
  const result = a * b;
  return result;
};

// single parameter (no parentheses needed)
const double = x => x * 2;
```

### key differences from regular functions:
- **no own `this`** — inherits `this` from the surrounding scope
- **cannot be used as constructors** — no `new` keyword
- **no `arguments` object** — use rest parameters instead

```js
const obj = {
  name: "olga",
  // ❌ arrow function — `this` is NOT the object
  greetArrow: () => console.log(this.name), // undefined

  // ✅ regular function — `this` IS the object
  greetRegular() { console.log(this.name); } // "olga"
};
```

---

## 🔓 object / array destructuring

destructuring lets you unpack values from arrays or objects into separate variables.

### object destructuring
```js
const user = { name: "olga", age: 25, city: "berlin" };

// without destructuring
const name = user.name;
const age = user.age;

// with destructuring
const { name, age, city } = user;

// renaming
const { name: userName, age: userAge } = user;

// default values
const { name, country = "unknown" } = user;
```

### array destructuring
```js
const colors = ["red", "green", "blue"];

const [first, second, third] = colors;
// first = "red", second = "green", third = "blue"

// skip values
const [, , last] = colors; // last = "blue"

// with rest
const [head, ...rest] = colors; // head = "red", rest = ["green", "blue"]
```

### in function parameters (very common in react)
```js
// instead of:
function greet(props) {
  console.log(props.name);
}

// destructure directly:
function greet({ name, age }) {
  console.log(name);
}
```

---

## 🌊 spread operator (...)

the spread operator expands an iterable (array, object) into individual elements.

### with arrays
```js
const a = [1, 2, 3];
const b = [4, 5, 6];

const combined = [...a, ...b]; // [1, 2, 3, 4, 5, 6]

// copy an array (shallow)
const copy = [...a];

// add to beginning/end
const withZero = [0, ...a]; // [0, 1, 2, 3]
```

### with objects
```js
const user = { name: "olga", age: 25 };

// copy + override
const updated = { ...user, age: 26 };
// { name: "olga", age: 26 }

// merge objects
const defaults = { theme: "light", lang: "en" };
const prefs = { theme: "dark" };
const config = { ...defaults, ...prefs };
// { theme: "dark", lang: "en" }
```

### rest parameters (the opposite — collects into an array)
```js
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3, 4); // 10
```

---

## 📦 modules (import / export)

### named exports
```js
// utils.js
export const PI = 3.14;
export function add(a, b) { return a + b; }

// main.js
import { PI, add } from './utils.js';
```

### default exports
```js
// logger.js
export default function log(msg) {
  console.log(msg);
}

// main.js
import log from './logger.js'; // can name it anything
import myLogger from './logger.js'; // also works
```

### named vs default
- **named**: must use exact name (or rename with `as`), can have many per file
- **default**: can name anything on import, only one per file

### ES modules vs CommonJS
```js
// ES modules (modern, browser + node 14+)
import { something } from './file.js';
export const value = 42;

// CommonJS (older, node.js traditional)
const { something } = require('./file.js');
module.exports = { value: 42 };
```
- **ES modules**: static imports, tree-shakeable, async loading
- **CommonJS**: dynamic imports, synchronous, no tree-shaking

---

## 📐 types, interfaces, generics (typescript)

### 🧩 types
```ts
let age: number = 25;
let name: string = Olga;
let isAdmin: boolean = true;
```
### 🧷 Interfaces
```ts
interface User {
    id: number;
    name: string;
}

const user: User = {id: 1; name: "Olga" };
```

### 🎁 Generics
```ts
function wrapInArray<T>(value: T): T[] {
    return [value];
}

const nums = wrapInArray<number>(5); // [5]
```

## 📦 objects, arrays, maps, sets
### Objects
- Key-value pairs
```js
const user = { name: "Alex", age: 22};
```
### Arrays
- Ordered collections
```js
const nunbers = [1, 2, 3];
```
### Maps
- Key-value with any type keys
```js
const maps = new Map();
map.set("a", 1);
```
### Sets
- Unique values only
```js
const set = new Set([1, 2, 2, 3]) // {1, 2, 3}
```
## 🏛️ classes, inheritance, this
```ts
class Animal {
    constructor(public name: string) {
        speak() {
            console.log(`${this.name} makes a sound`);
        }
    }

    class Dog extends Animal {
        speak();
        console.log(`${this.name} barks`)
    }
}

const d = new Dog("Rex");
d.speak(); // Rex barks
```

**this** keyword
- refers to the current context(object, function or global)
- arrow function don't have their own **this**

## common gotchas & tricks
❌ Don’t use == → use === for strict equality
✅ Use ?. (optional chaining) to safely access properties
✅ Use destructuring for cleaner code
✅ Prefer const by default, only use let when needed
❌ Avoid mutating objects/arrays directly – prefer immutability