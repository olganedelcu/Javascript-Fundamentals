# JavaScript Primitives

## Introduction

A primitive is a value that is not an object. It is the simple stuff: you store it, pass it around, compare it, print it.

Primitives are immutable(you do not change the value "inside" them - you create a new value).

## Primitive Data Types

There are 7 primitives:

### String

Basically, text. It is used for user IDs, messages, URLs, anythign text like. 

Example: 
```
const raw = "  olga.2002  " 

const username = raw.trim().toLowerCase() // we remove white spaces from begginning to end of string and make sure to make it lower case.

```

Funny case:
```
console.log("5" + 1) // 51, it is concatenated
console.log("5" - 1) // 4, string is converted to number lol

```

### Number

Integers and decimals. Used for prices, lists number, percentanges, maths operations...

Example: 
```
const grades = [9, 10, 5.9]
const sum = grades.reduce((sum, grades) => {sum + grades, 0})
```

### Boolean
True/False.

Used for permissions, toggles, flags, "is this valid?"

Example(form validation switch):
```
const email = "olganedelcuam@gmail.com";
const hasAt = email.include("@");
const hasDot = email.slipt("@")[1].include(".");

const isValidEmail = hasAt && hasDot;
if (!isValidEmail) console.log("It is not a valid email")
```

Truthiness(super practical):
```
const input = ""
if(!input) = console.log("disable submit button");
```

### Undefined
It is "not assigned" or "does not exit here".
Used for missing values, optional data not present, unninitialized variables.

Example:
```
const config = { theme: "dark"};
const lang = config.language; // undefined
console.log(lang ?? "en") // fallback "en"
```

### Null
"intentionally empty" "we know there is not value", deliberate reset.

Example: 
```
let selectedUserId = 42;

// later the user clicks on "clear selection"
selectedUserId = null;

if (selectedUserID === null) console.log("No user selected");
```

### BigInt
Very large integers.

Used for: huge IDs, financial ledger integers.

Example: Very large database IDs:
```
const id = 12234678765432567654n; 
const next = id + 1n; 

console.log(next); // outcome: 12234678765432567655n
```
### Symbol
Unique keys(to avoid name collisions)
Used for metadata, avoiding clashes with other code.

Example: 
```
const internalId = Symbol("internalId"); 

const user = { name: "Ana" };
user[internalId] = "x9a-23";

console.log(user.internalId); // undefined
console.log(user[internalId]);
```

## Primitive vs Object
Do not confuse primitives vs objects.

- Primitives behave like values:
```
let a = "hello";
let b = a;
b = "bye";

console.log(a) // "hello"
console.log(b) // "bye"
```
- Objects behave like references:
```
let a = { msg: "hello" };
let b = a;
b.msg = "bye";

console.log(a.msg) // "bye"

```
## Type Coercion
"Type check" cheat sheet:

```
typeof "x" // "string" text
typeof 10 // "number" normal numerical values
typeof true // "boolean" true/false decitions
typeof undefined // "undefined" missing/unset
typeof 10n // "bigint"  IDS, counters
typeof Symbol() // "symbol" - unique IDs to avoid collisions
typeof null // "object" -- weird JS 
```
