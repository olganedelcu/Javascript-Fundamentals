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