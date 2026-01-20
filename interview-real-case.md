# Interview Real Case Study

## 1. Coding Exercise - React "DateLabel/Timer" Component

### What they gave:
A React.js + TS component that:
- displays a start date
- displays the current date
- displays a human readable distance between them
- Uses
    - `useState`
    - `setInterval`
    - `date-fns`
    - `react-moment`

### Original Code (timer.tsx):
```typescript
import React, { FC, useCallback, useState } from 'react';
import { formatDistance } from 'date-fns';
import Moment from 'react-moment';

export const Datelabel: FC<any> = ({ title, countFromDate }) => {
  let startTime;
  startTime = countFromDate || new Date().getTime();
  title = title || 'Current time';
  const [date, setDate] = useState(new Date().getTime());
  const distance = formatDistance(startTime, new Date().getTime(), {
    addSuffix: true,
  });

  setInterval(() => {
    console.log('setting the date');
    setDate(new Date().getTime());
  });

  return (
    <div>
      {title} : {<Moment date={date} format="yyyy dddd D hh:mm:ss" />} started{' '}
      {distance}
    </div>
  );
};
```

## Problems they expected you to find

### 1. ❌ Reassigning props
```javascript
title = title || 'Current time';
```

**Issue**
- Props should be treated as immutable
- Reassigning props is a React anti-pattern

**What they wanted**
```javascript
const displayTitle = title ?? 'Current time';
```

**Knowledge tested**
- React immutability
- Props vs local variables
- `||` vs `??` semantics

### 2. ❌ Uninitialized let startTime
```javascript
let startTime;
startTime = countFromDate || new Date().getTime();
```

**Issues**
- `let` without initialization
- Imperative style instead of declarative
- Poor readability

**Expected**
```javascript
const startTime = countFromDate ?? Date.now();
```

**Knowledge tested**
- Functional style
- Readability
- Type safety

### 3. ❌ setInterval called during render
```javascript
setInterval(() => {
  setDate(Date.now());
});
```

**Critical issue**
- Runs on every render
- Creates infinite intervals
- Causes memory leaks
- Causes performance issues

**Expected fix**
```javascript
useEffect(() => {
  const interval = setInterval(() => {
    setDate(Date.now());
  }, 1000);

  return () => clearInterval(interval);
}, []);
```

**Knowledge tested**
- React render lifecycle
- Side effects
- `useEffect`
- Cleanup functions
- Memory leaks

### 4. ❌ Missing cleanup on unmount

**Problem**
- Interval continues running after component unmounts

**Expected**
- `clearInterval` inside `useEffect` cleanup

**Knowledge tested**
- Component lifecycle
- Browser APIs
- Resource management

### 5. ❌ Confusing state usage
```javascript
const [date, setDate] = useState(new Date().getTime());
```

**Observation**
- `startTime` never changes → should NOT be state
- `date` changes → SHOULD be state

**Expected understanding**
- Only put mutable, reactive data in state

**Knowledge tested**
- State vs derived values
- React mental model

### What this exercise was REALLY testing
- Can you read unfamiliar code
- Can you reason about side effects
- Can you identify performance bugs
- Can you explain WHY, not just fix

### ✅ Corrected Solution:
```typescript
import React, { FC, useEffect, useState } from 'react';
import { formatDistance } from 'date-fns';
import Moment from 'react-moment';

interface DateLabelProps {
  title?: string;
  countFromDate?: number;
}

export const DateLabel: FC<DateLabelProps> = ({ title, countFromDate }) => {
  const displayTitle = title ?? 'Current time';
  const startTime = countFromDate ?? Date.now();
  const [date, setDate] = useState(Date.now());

  useEffect(() => {
    const interval = setInterval(() => {
      setDate(Date.now());
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  const distance = formatDistance(startTime, date, {
    addSuffix: true,
  });

  return (
    <div>
      {displayTitle}: <Moment date={date} format="yyyy dddd D hh:mm:ss" />{' '}
      started {distance}
    </div>
  );
};
```

---

## 2. Coding Exercise #2 — TypeScript Transaction Types

### Original Code (types.ts):
```typescript
export type Transaction = {
  transactionType: 'SPORTS_BET' | 'DEPOSIT' | 'WITHDRAWAL' | 'CASINO_GAME';
  id: string;
  timestamp: number;
  amount: {
    currency: 'USD' | 'BTC';
    value: string;
  };
  details: {
    // always populated for SPORTS_BET transactions
    type?: string;
    homeTeam?: { name: string };
    awayTeam?: { name: string };
    homeScore?: number;
    awayScore?: number;

    // always populated for DEPOSIT transactions
    cardScheme?: 'VISA' | 'MASTERCARD';
    cardLastFour?: string;

    // always populated for CASINO_GAME transactions
    gameName?: string;
    gameId?: string;

    // always populated for WITHDRAWAL transactions
    fiatAmount?: {
      currency: 'USD' | 'BTC';
      value: string;
    };
    cryptoTransactionId?: string;
    cryptoTransactionLink?: string;
  };
};
```

**Problem they wanted you to solve**
- Make `details` depend on `transactionType`

**Right now:**
- All fields are optional
- Invalid states are allowed
- TypeScript cannot protect you

### ✅ Expected Solution Pattern
**Mapped Types + Generics**

```typescript
// Define specific detail types for each transaction type
type SportsBetDetails = {
  type: string;
  homeTeam: { name: string };
  awayTeam: { name: string };
  homeScore: number;
  awayScore: number;
};

type DepositDetails = {
  cardScheme: 'VISA' | 'MASTERCARD';
  cardLastFour: string;
};

type CasinoGameDetails = {
  gameName: string;
  gameId: string;
};

type WithdrawalDetails = {
  fiatAmount: {
    currency: 'USD' | 'BTC';
    value: string;
  };
  cryptoTransactionId: string;
  cryptoTransactionLink: string;
};

// Map transaction types to their specific details
type TransactionDetailsMap = {
  SPORTS_BET: SportsBetDetails;
  DEPOSIT: DepositDetails;
  WITHDRAWAL: WithdrawalDetails;
  CASINO_GAME: CasinoGameDetails;
};

// Generic transaction type that enforces correct details based on transactionType
export type Transaction<T extends keyof TransactionDetailsMap = keyof TransactionDetailsMap> = {
  id: string;
  timestamp: number;
  amount: {
    currency: 'USD' | 'BTC';
    value: string;
  };
  transactionType: T;
  details: TransactionDetailsMap[T];
};

// Usage examples:
// const sportsBet: Transaction<'SPORTS_BET'> = { ... }
// const deposit: Transaction<'DEPOSIT'> = { ... }
// Or without generic (union of all possible transactions):
// const anyTransaction: Transaction = { ... }
```

### Alternative: Discriminated Unions Approach

```typescript
type BaseTransaction = {
  id: string;
  timestamp: number;
  amount: {
    currency: 'USD' | 'BTC';
    value: string;
  };
};

type SportsBetTransaction = BaseTransaction & {
  transactionType: 'SPORTS_BET';
  details: SportsBetDetails;
};

type DepositTransaction = BaseTransaction & {
  transactionType: 'DEPOSIT';
  details: DepositDetails;
};

type CasinoGameTransaction = BaseTransaction & {
  transactionType: 'CASINO_GAME';
  details: CasinoGameDetails;
};

type WithdrawalTransaction = BaseTransaction & {
  transactionType: 'WITHDRAWAL';
  details: WithdrawalDetails;
};

export type Transaction =
  | SportsBetTransaction
  | DepositTransaction
  | WithdrawalTransaction
  | CasinoGameTransaction;
```

**Knowledge tested**
- Discriminated unions
- Conditional typing
- Generics
- Modeling real-world data
- Preventing invalid states

---

## 3. Fetch + Generics Question

**What they asked**
- How would you type a fetch that returns a specific transaction?

**Expected answer:**
```typescript
async function fetchTransaction<T extends TransactionType>(): Promise<Transaction<T>> {
  const res = await fetch('/api/transaction');
  return res.json();
}
```

**Knowledge tested**
- Generics in async functions
- API typing
- End-to-end type safety

---

## 4. Conceptual Questions Asked

### React / Next.js

**Questions**
- Why use Next.js instead of React?
- What problem does SSR solve?
- What is SSG?

**Expected knowledge**
- SEO
- Server-side rendering
- Static generation
- HTML vs JS execution

### GraphQL vs REST

**Questions**
- When should you NOT use GraphQL?
- Why is GraphQL useful?
- Advantages over REST?

**Expected answers**
- Overfetching / underfetching
- Client-driven queries
- Schema + documentation
- Performance tradeoffs
- Not good for very simple APIs

### Databases (Postgres / MySQL)

**Questions**
- How do you debug a slow SQL query?
- What is `EXPLAIN ANALYZE`?
- What is an index?
- How does binary search relate to indexes?

**Expected knowledge**
- Query plans
- Index scans vs sequential scans
- B-tree indexes
- Complexity reduction
- When indexes help / hurt

---

## 5. Meta Skills They Were Evaluating

They were NOT just testing code.

They were testing:
- 🧠 **Reasoning**
- 🗣 **Communication**
- 🔍 **Debugging thought process**
- 📐 **Design decisions**
- ⚠️ **Edge cases**
- 🧹 **Cleanup & performance awareness**

---

## Interview Talking Points - Quick Reference

### React – Props, State, and Effects

**❓ Why is reassigning props a bad practice?**

**Answer:**
Props are immutable by design. Reassigning them breaks React's one-way data flow and makes components harder to reason about. Instead, I derive a new local variable from props when I need a default value.

**❓ When should something be in state?**

**Answer:**
Only values that change over time and should trigger a re-render belong in state. Derived or static values should be computed directly during render.

**❓ Why is setInterval inside the component body a problem?**

**Answer:**
Because it runs on every render, creating multiple intervals and causing memory leaks. Side effects like intervals must be handled inside `useEffect`.

**❓ How do you properly clean up side effects?**

**Answer:**
By returning a cleanup function from `useEffect`, which runs when the component unmounts or dependencies change.

---

### Next.js

**❓ Why use Next.js instead of plain React?**

**Answer:**
Next.js adds server-side rendering and static generation, which improves SEO, initial page load, and performance. It also provides routing, API routes, and optimized builds out of the box.

---

### GraphQL vs REST

**❓ Why use GraphQL?**

**Answer:**
It allows clients to request exactly the data they need, reducing overfetching and underfetching. It also provides a strongly typed schema and built-in documentation.

**❓ When should you NOT use GraphQL?**

**Answer:**
For very simple APIs, internal services, or when caching and CDN support are critical, REST is often simpler and more efficient.

---

### Databases

**❓ How do you debug a slow SQL query?**

**Answer:**
I use `EXPLAIN ANALYZE` to inspect the query plan, check for sequential scans, missing indexes, and inefficient joins.

**❓ What is an index?**

**Answer:**
An index is a data structure (usually a B-tree) that allows the database to locate rows efficiently, similar to binary search on sorted data.

---

## 6. Bonus Exercise — Custom Timeout Function

### Problem (custom-timeout.ts):
```typescript
// timeout(() => console.log('Hello!'), 2000);

export function timeout(fn, ms) {}
```

### Goal:
Implement a `setTimeout` replacement that:
- Executes a function after a delay
- Returns a cancel function
- Works without using `setTimeout`

### ✅ Solution 1: Using `setInterval`
```typescript
export function timeout(fn: () => void, ms: number): () => void {
  let cancelled = false;
  const start = Date.now();

  const interval = setInterval(() => {
    if (cancelled) {
      clearInterval(interval);
      return;
    }

    if (Date.now() - start >= ms) {
      clearInterval(interval);
      fn();
    }
  }, 10); // Check every 10ms

  // Return cancel function
  return () => {
    cancelled = true;
    clearInterval(interval);
  };
}
```

### ✅ Solution 2: Using `requestAnimationFrame` (Browser only)
```typescript
export function timeout(fn: () => void, ms: number): () => void {
  let cancelled = false;
  const start = Date.now();

  function check() {
    if (cancelled) return;

    if (Date.now() - start >= ms) {
      fn();
    } else {
      requestAnimationFrame(check);
    }
  }

  requestAnimationFrame(check);

  return () => {
    cancelled = true;
  };
}
```

### ✅ Solution 3: Using recursion + setImmediate (Node.js)
```typescript
export function timeout(fn: () => void, ms: number): () => void {
  let cancelled = false;
  const start = Date.now();

  function check() {
    if (cancelled) return;

    if (Date.now() - start >= ms) {
      fn();
    } else {
      setImmediate(check);
    }
  }

  check();

  return () => {
    cancelled = true;
  };
}
```

### Usage Example:
```typescript
const cancel = timeout(() => console.log('Hello!'), 2000);

// To cancel:
// cancel();
```

**Knowledge tested:**
- Understanding of event loop
- Alternative timing mechanisms
- Closure for cancellation
- Recursion patterns
- Browser vs Node.js APIs
