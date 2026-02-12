### What is a generic function in TS and can you write one?

🔁 a generic function is a way to write code that is type-flexible —
without using any.

⚙️ meaning: you can provide not only regular arguments, but also **typed arguments** — and the function still works for different data types.

```ts
function myNewFunction<T>(one: T): T {
  return one;
}
```
💡 here, T is the generic type argument.
we can call this function like:
```ts
const result = myNewFunction<number>(5);
```
✅ and now result is inferred to be of type number.

🛑 but if you try:
```ts
const result = myNewFunction<number>("string");
```
❌ you get an **error** — because **"string" is not a number, and TS catches that**.

📦 that's why they’re called **generic functions** — you declare a type variable like T,
and then use it consistently across parameters and return types.
this helps you reuse logic, without sacrificing type safety.

#### ❓ what is an as const in typescript? how is it different from normal const?

🛡️ when you use as const, you are telling typescript to treat the entire object as deeply immutable.
🔒 this means you cannot modify any part of it — not even nested properties.
```ts
const config = {
    host: "whastsf";
    port:45 ;
    auth: {
        client: "ad",
    }
} as const;
```
if you try
```ts
config.host = "whatever"; // ❌ not possible
config.auth.client = "whatever"; // ❌ we can not change it
```
you will get an error.

📦 every nested property becomes immutable.

💬 in js, you could also freeze the object:
```ts
Object.freeze(config);
```
but ⚠️ this only works at runtime — and it only freezes the first level,
not the nested ones.

🛠️ as const is evaluated at build time, not runtime - it’s a typescript-only tool for locking objects.

### What is the difference between type and interface in Typescript? When would you use one or the other

we could interchange both for most part, the only difference at build time interfaces can be **merged**, types must be **unique**.

this means that you can declare **the same interface multiple times**, and TS will **merge** them automatically.

but if you declare the **same type name twice**, it will throw an **error**.

💡 so in a **codebase**, you usually go for one of them and stick with it for **consistency**.

#### when use them?
if you're building a library, where you want people to extend certain types (e.g. Express.js with Request object),
it's better to use an **interface** — and TS will merge them behind the scenes if a person expands that interface.

i usually use type to define domain entities — e.g. ecommerce Product, Price, etc and use interface for auxiliary shapes like:

- **props** of a React component
- **input** params of a function

#### What is a type guard in TS? When use it
it's a way of writing a function that **filters** or guards a **specific type** from **a union type**.
```ts
type Fruit = Pineapple | Banana;
```
you can write a function like:
```ts
function isPineapple(fruit: Fruit): fruit is Pineapple {
  return (fruit as Pineapple).spiky !== undefined;
}
```
this allows you to **narrow down** the type safely.
so inside that condition, TS knows you’re working with a **Pineapple** —
and you get **autocompletion**, type safety, and error checking.
use it whenever you're dealing with **union types**, and you need to work with one specific subtype.

---

## arrays & tuples

### arrays
```ts
const names: string[] = ["olga", "ana", "maria"];
const ids: Array<number> = [1, 2, 3];
```

### tuples
a tuple is a fixed-length array where each position has a specific type.
```ts
const user: [string, number] = ["olga", 25];
// user[0] is string, user[1] is number

// useful for things like coordinates, key-value pairs, etc.
const coordinates: [number, number] = [40.7, -74.0];

// named tuples (for readability)
type HttpResponse = [status: number, body: string];
const res: HttpResponse = [200, '{"ok": true}'];
```

---

## union types

a union type means a value can be **one of several types**.
```ts
type Status = "loading" | "success" | "error";
type Id = string | number;

function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // TS knows it's string here
  } else {
    console.log(id.toFixed(2)); // TS knows it's number here
  }
}
```

---

## intersection types

an intersection type combines multiple types into one — the result must satisfy **all** of them.
```ts
type HasName = { name: string };
type HasAge = { age: number };

type Person = HasName & HasAge;

const user: Person = { name: "olga", age: 25 }; // must have both
```
useful for composing types from smaller building blocks.

---

## literal types

a literal type is a type that represents **one specific value**.
```ts
type Direction = "up" | "down" | "left" | "right";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type StatusCode = 200 | 404 | 500;

// TS will only allow these exact values:
let dir: Direction = "up"; // ✅
dir = "diagonal"; // ❌ error
```

---

## optional properties (?)

marks a property as not required.
```ts
interface User {
  name: string;
  email?: string; // optional
}

const user1: User = { name: "olga" }; // ✅ valid
const user2: User = { name: "olga", email: "olga@test.com" }; // ✅ also valid
```

---

## readonly properties

marks a property as immutable after creation.
```ts
interface Config {
  readonly apiUrl: string;
  readonly port: number;
}

const config: Config = { apiUrl: "https://api.example.com", port: 3000 };
config.apiUrl = "something"; // ❌ error - cannot reassign readonly
```

---

## index signatures

when you don't know the property names ahead of time, but know the shape.
```ts
interface StringMap {
  [key: string]: string;
}

const headers: StringMap = {
  "Content-Type": "application/json",
  "Authorization": "Bearer token123",
};

// also works with number keys:
interface NumberMap {
  [index: number]: string;
}

const colors: NumberMap = { 0: "red", 1: "blue" };
```

---

## generic constraints (<T extends Something>)

you can restrict what types a generic accepts.
```ts
// T must have a .length property
function logLength<T extends { length: number }>(item: T): void {
  console.log(item.length);
}

logLength("hello"); // ✅ string has .length
logLength([1, 2, 3]); // ✅ array has .length
logLength(42); // ❌ number does NOT have .length

// T must be a key of an object
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "olga", age: 25 };
getProperty(user, "name"); // ✅ "olga"
getProperty(user, "email"); // ❌ "email" is not a key of user
```

---

## utility types (extremely common in codebases)

### Partial\<T\>
makes all properties optional.
```ts
interface User {
  name: string;
  email: string;
  age: number;
}

// useful for update functions where you only send changed fields
function updateUser(id: string, updates: Partial<User>) {
  // updates can be { name: "new" } or { age: 30 } or all of them
}

updateUser("123", { name: "olga" }); // ✅ only name
```

### Required\<T\>
makes all properties required (opposite of Partial).
```ts
type RequiredUser = Required<Partial<User>>; // back to all required
```

### Pick\<T, Keys\>
creates a type with only specific properties.
```ts
type UserPreview = Pick<User, "name" | "email">;
// { name: string; email: string }
```

### Omit\<T, Keys\>
creates a type without specific properties.
```ts
type UserWithoutAge = Omit<User, "age">;
// { name: string; email: string }
```

### Record\<K, V\>
creates a type with keys of type K and values of type V.
```ts
type Role = "admin" | "user" | "guest";

const permissions: Record<Role, string[]> = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};

// also great for dictionaries:
type ApiResponses = Record<string, { status: number; data: unknown }>;
```

---

## typing API work (very important for backend/integration)

### typing request/response objects
```ts
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: string;
  name: string;
  email: string;
}

// the response is typed end-to-end
const response: ApiResponse<User> = {
  data: { id: "1", name: "olga", email: "olga@test.com" },
  status: 200,
  message: "success",
};
```

### typing nested JSON
```ts
interface Address {
  street: string;
  city: string;
  country: string;
}

interface Company {
  name: string;
  address: Address;
  employees: Employee[];
}

interface Employee {
  id: number;
  name: string;
  role: string;
}

// now deeply nested data is fully typed
```

### typing function return values
```ts
// explicit return type
async function fetchUsers(): Promise<User[]> {
  const res = await fetch("/api/users");
  return res.json();
}

// void for functions that don't return
function logMessage(msg: string): void {
  console.log(msg);
}

// never for functions that never return
function throwError(msg: string): never {
  throw new Error(msg);
}
```

---

## narrowing & guards (handling external API data)

### typeof
```ts
function process(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // TS knows it's string
  }
  return value * 2; // TS knows it's number
}
```

### instanceof
```ts
function handleError(error: unknown) {
  if (error instanceof Error) {
    console.log(error.message); // TS knows it's Error
  }
}
```

### discriminated unions
```ts
type Success = { status: "success"; data: string };
type Failure = { status: "error"; error: string };
type Result = Success | Failure;

function handle(result: Result) {
  if (result.status === "success") {
    console.log(result.data); // TS knows it's Success
  } else {
    console.log(result.error); // TS knows it's Failure
  }
}
```

useful for handling external APIs that can return different shapes.

