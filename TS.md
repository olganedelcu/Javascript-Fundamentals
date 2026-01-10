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
use it whenever you’re dealing with **union types**, and you need to work with one specific subtype.

