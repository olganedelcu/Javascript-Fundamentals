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
