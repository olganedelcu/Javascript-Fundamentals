# React Fundamentals & Lifecycle

## what is react?

**react is a library, not a framework**
- library: provides specific functionality (UI rendering)
- framework: provides entire application structure

**core concepts:**
- **declarative UI** - describe what the UI should look like, react handles the how
- **components as pure functions** - given the same inputs (props), return the same output
- **one-way data flow** - data flows down from parent to child via props

---

## props

### what props are
- immutable inputs to a component
- passed from parent to child
- read-only by design

### props immutability

**❌ NEVER reassign props:**
```typescript
export const Component = ({ title }) => {
  title = title || 'Default'; // BAD - mutates props
  // ...
}
```

**issue:**
- mutates props (breaks react assumptions)
- violates one-way data flow
- makes components harder to reason about
- `||` treats `0`, `false`, `''` as falsy

**✅ derive a new variable instead:**
```typescript
export const Component = ({ title }) => {
  const displayTitle = title ?? 'Default'; // GOOD
  // ...
}
```

### default values: `??` vs `||`

**`||` - fallback on ANY falsy value**
- triggers on: `0`, `false`, `''`, `null`, `undefined`
- use when: only want to exclude falsy values

**`??` - fallback ONLY on `null` or `undefined`**
- triggers on: `null`, `undefined`
- use when: `0`, `false`, `''` are valid values

**examples:**
```typescript
const count = 0;
count || 10  // → 10 (0 is falsy)
count ?? 10  // → 0  (0 is not null/undefined)

const name = '';
name || 'Guest'  // → 'Guest' ('' is falsy)
name ?? 'Guest'  // → ''      ('' is not null/undefined)
```

**rule of thumb:** use `??` when `0`, `false`, or `''` are valid values.

### props vs derived values

**props** - passed from parent
**derived values** - calculated from props or state

```typescript
const Component = ({ count }) => {
  // count is a prop
  const doubled = count * 2; // derived value (don't put in state!)
  return <div>{doubled}</div>;
}
```

---

## state

### what state is
- data that changes over time
- triggers re-renders when updated
- local to the component

### when to use `useState`

**use state when:**
- ✅ value changes over time
- ✅ change should trigger a re-render
- ✅ value is not derived from props

### when NOT to use state

**don't use state for:**
- ❌ values derived from props
- ❌ initialization-only values
- ❌ constants

**example:**
```typescript
// ❌ BAD - startTime never changes
const [startTime, setStartTime] = useState(Date.now());

// ✅ GOOD - just a constant
const startTime = Date.now();

// ✅ GOOD - date changes over time
const [date, setDate] = useState(Date.now());
```

### state updates trigger re-renders

```typescript
const [count, setCount] = useState(0);
setCount(1); // component re-renders with new count
```

### derived state vs stored state

**stored state** - explicit `useState`
**derived state** - calculated during render

```typescript
const [count, setCount] = useState(0);
const doubled = count * 2; // derived - recalculates on every render
```

**avoid unnecessary state:**
```typescript
// ❌ BAD
const [count, setCount] = useState(0);
const [doubled, setDoubled] = useState(0);

useEffect(() => {
  setDoubled(count * 2);
}, [count]);

// ✅ GOOD
const [count, setCount] = useState(0);
const doubled = count * 2;
```

---

## component lifecycle

a react function component goes through three phases:

1. **mount** - component appears for the first time
2. **update** - component re-renders due to state or prop changes
3. **unmount** - component is removed from the UI

### how lifecycle maps to `useEffect`

in function components, side effects are controlled with `useEffect`:

```typescript
useEffect(() => {
  // side effect runs AFTER render

  return () => {
    // cleanup runs BEFORE next effect or on unmount
  };
}, [dependencies]);
```

### how `useEffect` works internally

**mount** → effect runs after DOM paint
**update** → cleanup runs → effect runs again (if deps changed)
**unmount** → cleanup runs once

---

## react rendering vs committing to the DOM

**render phase:**
- react calls your component function
- calculates what changed
- can be interrupted or retried

**commit phase:**
- react applies changes to the DOM
- effects run AFTER commit

```
render → commit to DOM → paint → effects
```

---

## useEffect

### what `useEffect` is

a hook for performing side effects in function components.

### why side effects don't belong in render

**render must be pure:**
- ❌ no timers
- ❌ no subscriptions
- ❌ no mutations
- ❌ no side effects

**why?**
- react can render multiple times
- render may be interrupted or retried
- side effects during render cause memory leaks & duplication

### what "render" actually means in react

when react calls your component function:

```typescript
function MyComponent() {
  // ← this entire body is "render"
  // side effects here = BAD

  return <div />;
}
```

**when we say "side effects don't belong in render", we mean:**
you should not run side effects directly in the component body.
they must live inside `useEffect` (or other effect hooks).

**that's not a style rule** - it's because of how react works under the hood.

### dependency array behavior

```typescript
useEffect(() => {
  // ...
}, []); // empty array → runs once on mount

useEffect(() => {
  // ...
}, [count]); // runs when count changes

useEffect(() => {
  // ...
}); // no array → runs after EVERY render (usually wrong)
```

### effect cleanup function

```typescript
useEffect(() => {
  const id = setInterval(() => {
    console.log('tick');
  }, 1000);

  // cleanup function
  return () => {
    clearInterval(id);
  };
}, []);
```

### when cleanup runs

- before the effect runs again (on update)
- when the component unmounts

### why cleanup matters

- ✅ prevents memory leaks
- ✅ prevents duplicate logic after re-renders
- ✅ ensures predictable lifecycle behavior

---

## side effects

### what counts as a side effect

- network requests (fetch, axios)
- timers (`setTimeout`, `setInterval`)
- subscriptions (WebSocket, EventSource)
- event listeners (window, document)
- DOM manipulation
- logging
- localStorage/sessionStorage

### why side effects must be isolated

side effects during render can:
- create memory leaks
- cause duplicate operations
- make components unpredictable
- break react's rendering model

---

## setInterval in react (deep dive)

### what `setInterval` is (JavaScript)

- a browser API (not react)
- takes a callback and a delay (ms)
- returns an interval ID

```javascript
const id = setInterval(callback, delay);
clearInterval(id);
```

### why `setInterval` inside render is dangerous

```typescript
// ❌ DANGEROUS
function Component() {
  setInterval(() => {
    console.log('tick');
  }, 1000);

  return <div />;
}
```

**what happens:**
1. render runs → interval created
2. state update → render again → **another interval**
3. leads to:
   - multiple intervals running
   - memory leaks
   - performance degradation

### using `setInterval` inside `useEffect`

```typescript
// ✅ CORRECT
useEffect(() => {
  const intervalId = setInterval(() => {
    setDate(Date.now());
  }, 1000);

  return () => {
    clearInterval(intervalId);
  };
}, []); // run once on mount
```

### cleaning up intervals

**why cleanup is critical:**
- interval continues running even after unmount
- multiple intervals pile up
- causes memory leaks and performance issues

---

## re-rendering

### what causes a re-render

- state changes (`setState`)
- props change (parent re-renders)
- context changes
- force update (rare, avoid)

### avoiding unnecessary re-renders

- keep state close to where it's used
- split large components
- use proper dependency arrays
- memoization (when needed)

### state granularity

**good:**
```typescript
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
```

**bad:**
```typescript
const [user, setUser] = useState({ firstName: '', lastName: '' });
// changing firstName re-renders even if only lastName is displayed
```

### pure render functions

given the same props and state, should always return the same JSX.

---

## JSX & imports

### JSX transformation

JSX is transformed to `React.createElement` calls.

**before react 17:**
```typescript
import React from 'react'; // required
```

**react 17+:**
```typescript
// no react import needed for JSX
import { useState } from 'react';
```

### why import react is no longer required

react 17+ uses a new JSX transform that doesn't require `React` in scope.

### when react import is still needed

```typescript
import React from 'react';

// when using React.Something:
React.memo(Component);
React.forwardRef(...);
React.lazy(...);
```

---

## modern JavaScript features

### Date.now() vs new Date().getTime()

**old way (pre-ES5, before 2009):**
```typescript
var timestamp = new Date().getTime(); // ❌ legacy
```

**how it works:**
1. creates Date object in memory
2. calls `getTime()` method
3. returns number
4. garbage collector cleans up the Date object

**performance:** slower - more memory allocation, GC cleanup required

**modern way (ES5+):**
```typescript
const timestamp = Date.now(); // ✅
```

**performance:** faster - returns timestamp directly, no object allocation

### other important JS concepts

**closures in hooks:**
```typescript
useEffect(() => {
  const value = count; // closure captures current count
  setTimeout(() => {
    console.log(value); // uses captured value
  }, 1000);
}, [count]);
```

**referential equality:**
```typescript
{} === {}  // false (different references)
[1] === [1]  // false
```

**immutability in JavaScript:**
```typescript
const obj = { count: 0 };
obj.count = 1; // legal - const only prevents reassignment
obj = {}; // error - can't reassign
```

**const vs let:**
- `const` - can't reassign, but can mutate
- `let` - can reassign

---

## common react mistakes

### ❌ side effects in render
```typescript
// BAD
function Component() {
  setInterval(...); // creates new interval every render
  return <div />;
}
```

### ❌ reassigning props
```typescript
// BAD
const Component = ({ title }) => {
  title = title || 'Default';
}
```

### ❌ using state for constants
```typescript
// BAD
const [startTime, setStartTime] = useState(Date.now());
// startTime never changes - shouldn't be state
```

### ❌ forgetting effect cleanup
```typescript
// BAD
useEffect(() => {
  setInterval(...);
  // missing cleanup!
}, []);
```

### ❌ overusing global state
keep state as local as possible.

---

## key mental models

### render is not lifecycle
- render is just a function call
- it calculates what UI should look like
- it doesn't DO anything

### effects run after render
```
render → commit to DOM → paint → effects
```

### cleanup prevents leaks
always clean up:
- timers
- subscriptions
- event listeners
- network requests (abort controllers)

### react may render more than you expect
- strict mode renders twice in development
- react may discard renders
- write resilient code

### think in data flow, not events
- don't think "when button clicks, update state"
- think "state determines UI"

---

## complete example: DateLabel component

```typescript
import { FC, useEffect, useState } from 'react';
// in modern react you do not need to import React unless you are using React.Something
import { formatDistance } from 'date-fns';
import Moment from 'react-moment';

interface DateLabelProps {
  title?: string;
  countFromDate?: number;
}

const DateLabel: FC<DateLabelProps> = ({ title, countFromDate }) => {
  // state: date changes over time, triggers re-renders
  const [date, setDate] = useState(Date.now());

  // constant: startTime never changes, NOT state
  // before JS ES5, Date.now() did not exist
  // old way: new Date().getTime()
  // - slower performance
  // - 1. creates Date object in memory
  // - 2. calls getTime() method
  // - 3. returns number
  // - 4. garbage collector cleans up the Date object
  // old codebases written before 2009 used this
  const startTime = countFromDate ?? Date.now();

  // derived value: use ?? to allow 0, '', false as valid values
  // do not reassign props!
  const displayTitle = title ?? 'Current time';

  // derived value: recalculates when date changes
  // even if date updates, distance recalculates because it depends on date
  const distance = formatDistance(startTime, date, {
    addSuffix: true,
  });

  // side effect: setInterval must be in useEffect
  useEffect(() => {
    // setInterval is a JS function
    // takes: callback, delay
    // returns: id number
    const intervalId = setInterval(() => {
      console.log('setting the date');
      setDate(Date.now());
    }, 1000);

    // cleanup: prevents memory leaks
    return () => {
      clearInterval(intervalId);
    };
  }, []); // empty array → run once on mount

  return (
    <div>
      {displayTitle}: <Moment date={date} format="yyyy dddd D hh:mm:ss" />{' '}
      started {distance}
    </div>
  );
};

export default DateLabel; // PascalCase convention for components
```

### why this code works

1. **state used correctly** - `date` changes over time
2. **constants not in state** - `startTime` is just a value
3. **props not reassigned** - derived to `displayTitle`
4. **side effect isolated** - `setInterval` in `useEffect`
5. **cleanup provided** - `clearInterval` prevents leaks
6. **derived values** - `distance` recalculates on render
7. **modern JS** - `Date.now()`, `??`, functional style

---

## summary checklist

- [ ] understand react is a library, not a framework
- [ ] never reassign props
- [ ] use `??` for default values when `0`, `''`, `false` are valid
- [ ] only use state for values that change over time
- [ ] keep side effects in `useEffect`, not render
- [ ] always provide cleanup functions for timers/subscriptions
- [ ] understand the three lifecycle phases: mount, update, unmount
- [ ] know when cleanup runs (before next effect, on unmount)
- [ ] use `Date.now()` instead of `new Date().getTime()`
- [ ] think in data flow, not events
- [ ] render must be pure
- [ ] effects run after render and commit
