# ⏳ javascript: async, promises & more

## 🤔 is javascript synchronous?

yes - **javascript is a synchronous, single-threaded language**,
it runs **one thing at a time**, line by line.

but... we can **mimic asynchronous behaviour** using: 
- **callbacks**
- **promises**
- **async/await**

these tools allow us to:
- wait for data (like from APIs)
- handle delays (like timers)
- avoid blocking the main thread

---

## 🪝 callbacks

a **callback** is a function passed into another function to be run later.

```js
function fetchData(callback) {
    setTimeout(() => {
        callback('data received!');
    },1000)
}

fetchData((result) => {
    console.log(result);
});
```
🧠 but **callbacks** can get messy - known as "callback hell":

```js
doThing(() => {
    doNext() => {
        doAnother(() =>{
            ...
        })
    }
})
```
## 💭 promises
a **promise** is javascript object that represents a value you **don't have yet**, but will get in the **future**(or maybe not).

it's like saying "i promise you something later"

🧱 **promise has 3 possible states:**
- `pending` – still waiting  
- `fulfilled`- got the value
- `rejected` - something went wrong

you can **.then()** for success, **.catch()** for erros: 
```js
    fetchData
    .then((res) => console.log(res))
    .catch((err) => console.log(err));
```
## ⚡ async / await
- syntactic sugar over promises 🚀
- ✅ makes async code look synchronous
- 📦 await can only be used inside an async function
```js
async function getData() {
    try {
        const res = await fetchData();
        console.log(res);
    } catch (err) {
        console.log(err);
    }
}
```

#### what does Async functions return and why?
⚙️ async await is a **syntax sugar** for **promises** and whenever you write async in front of a function, it will basically **wrap** **that code in a promise** and return a promise that will **resolve** with the return of that function. 
🧠 so whenever you have an async function, that function will return a promise that will end up resolving with whatever the return of that function is.
example:

```js
import {useEffect, useState } from "react";


function Characters() {
    const [people, setPeople ] = useState([]);
    
    async function getPeople(){
       try {
            const data = await fetch("https://swapi.dev/api/people");
            const response = await data.json();
            setPeople(response.results);  
       }  catch(err) {
            console.log(err.message);
       }
    }
    
    useEffect(() => {
        getPeople();
    }, [])

    return <>
    {people.map((character, index) => (
        return <p key={index}>{character.name}
    )
        </p>
    )}
    </>
}

export default Characters;

```

---

## 🔄 Promise.all, Promise.race, Promise.allSettled

### Promise.all
runs multiple promises **in parallel**, resolves when **all** succeed, rejects if **any** fail.
```js
const [users, posts, comments] = await Promise.all([
  fetch("/api/users").then(r => r.json()),
  fetch("/api/posts").then(r => r.json()),
  fetch("/api/comments").then(r => r.json()),
]);
// all three fetched in parallel — much faster than sequential
```
⚠️ if one fails, the whole thing rejects.

### Promise.allSettled
runs multiple promises in parallel, waits for **all** to finish (success or failure).
```js
const results = await Promise.allSettled([
  fetch("/api/users"),
  fetch("/api/failing-endpoint"),
]);

results.forEach(result => {
  if (result.status === "fulfilled") {
    console.log("success:", result.value);
  } else {
    console.log("failed:", result.reason);
  }
});
```
useful when you want **all results** even if some fail.

### Promise.race
resolves/rejects with the **first** promise to settle.
```js
const result = await Promise.race([
  fetch("/api/data"),
  new Promise((_, reject) =>
    setTimeout(() => reject(new Error("timeout")), 5000)
  ),
]);
// if fetch takes longer than 5s, the timeout wins
```

---

## 🔁 event loop basics

javascript is **single-threaded** — it can only do one thing at a time.
but it can handle async operations using the **event loop**.

### how it works:
```
1. call stack — executes synchronous code, one at a time
2. web APIs / node APIs — handle async operations (timers, fetch, I/O)
3. callback queue (task queue) — holds callbacks ready to run
4. microtask queue — holds promise callbacks (.then, async/await)
5. event loop — moves tasks from queues → call stack when stack is empty
```

### execution order:
```js
console.log("1"); // synchronous — runs first

setTimeout(() => console.log("2"), 0); // macro task — runs last

Promise.resolve().then(() => console.log("3")); // microtask — runs second

console.log("4"); // synchronous — runs after "1"

// output: 1, 4, 3, 2
```

**key rule:** microtasks (promises) always run before macro tasks (setTimeout, setInterval).

---

## ⏱️ handling API timeouts

### using Promise.race for timeout
```js
async function fetchWithTimeout(url, timeoutMs = 5000) {
  const controller = new AbortController();

  const timeout = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, { signal: controller.signal });
    clearTimeout(timeout);
    return await response.json();
  } catch (error) {
    clearTimeout(timeout);
    if (error.name === "AbortError") {
      throw new Error(`request timed out after ${timeoutMs}ms`);
    }
    throw error;
  }
}
```

### using AbortController (modern approach)
```js
const controller = new AbortController();

// cancel after 3 seconds
setTimeout(() => controller.abort(), 3000);

try {
  const res = await fetch("/api/data", { signal: controller.signal });
  const data = await res.json();
} catch (err) {
  if (err.name === "AbortError") {
    console.log("request was cancelled");
  }
}
```

---

## 🔁 retry logic

when an API call fails, you might want to retry it a few times before giving up.

### simple retry
```js
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (error) {
      console.log(`attempt ${i + 1} failed: ${error.message}`);
      if (i === retries - 1) throw error; // last attempt — give up
    }
  }
}
```

### retry with exponential backoff
```js
async function fetchWithBackoff(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (error) {
      if (i === retries - 1) throw error;
      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s...
      console.log(`retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```
exponential backoff is important for production APIs — it prevents hammering a failing server.

---

### What is a pure function and when do we use it?
A pure function is very useful when using **functional programming**. Why? Because it really **does not modify anything else** — it has **no side effects**. 🌱

You really just **give it an input**, and it gives you an **output**, and there is no other change to:

- any global state of the system 🌐,
- or any **calls** outside the system (like APIs, localStorage, DBs, etc.)

This is extremely good because they are very **deterministic** ✅ — so you always know:
- given an input → you always get the same output.

they're commonly used in **functional programming** and in most J**avaScript codebases**, because it makes things simpler, more predictable, and easier to **test**.