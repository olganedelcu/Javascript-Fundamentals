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
```js
function outer() {
    let count = 0;
    return function inner() {
        count++;
        console.log(count);
    };
}

const inc = outer();
inc();
inc();
```
- used for data privacy, stateful functions, and async logic.

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

## common gotchas & tricks