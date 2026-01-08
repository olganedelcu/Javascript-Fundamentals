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



## 📐 types, interfaces, generics (typescript)

## 📦 objects, arrays, maps, sets

## 🏛️ classes, inheritance, this

## common gotchas & tricks