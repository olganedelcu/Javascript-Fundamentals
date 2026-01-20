# graphql vs rest 🧙‍♀️

##### when should you NOT use graphql? 🧙‍♀️
##### when is graphql a bad idea? 🧙‍♀️
##### why would you use graphql? 🧙‍♀️
##### what do you like about graphql?
##### what problem does graphql solve?
##### how is graphql different from rest?
##### what about documentation? 📚
##### what about data overfetching? 📦

---

# graphql vs rest – technical rationale and trade-offs

## 1. what graphql is 🧙‍♀️

graphql is a query language and runtime for apis created by facebook (meta) and released publicly in 2015. unlike rest, which exposes multiple endpoints, graphql exposes a single endpoint where clients specify exactly what data they need.

graphql is not a database and not a replacement for backend logic. it is an api layer that sits between the client and the underlying data sources.

---

## 2. why graphql was created ✨

graphql was created to solve problems that emerged at facebook scale, particularly:

- rapidly evolving product requirements
- multiple clients (web, mobile, internal tools)
- inefficient data fetching with rest

in rest-based systems, clients often needed to:

- call multiple endpoints to build a single screen
- receive more data than needed (overfetching)
- receive too little data and make additional requests (underfetching)

graphql was designed to give clients control over data shape, while allowing the server to evolve independently.

---

## 3. the core problem graphql solves 🎯

the main problem graphql addresses is **inefficient and inflexible data fetching**.

### with rest:

- endpoints define response shape
- clients adapt to backend decisions
- changes often require new endpoints or versions

### with graphql:

- clients define exactly what fields they need
- a single request can fetch nested, related data
- backend changes do not necessarily break clients

this is particularly valuable for complex uis and mobile applications.

---

## 4. how graphql is different from rest 🔄

### rest:

- multiple endpoints
- fixed response shapes
- strong alignment with http semantics
- simpler caching

### graphql:

- single endpoint
- flexible, client-defined queries
- strongly typed schema
- requires more server-side complexity

graphql shifts complexity from the client and api surface into the schema and resolver layer.

---

## 5. overfetching and underfetching 📊

### overfetching (rest):

- client receives data it does not need
- increases payload size
- impacts performance on slow networks

### underfetching (rest):

- client needs multiple requests to build a view
- increases latency
- more coordination between frontend and backend

graphql eliminates both by allowing clients to request only what they need, in a single query.

---

## 6. documentation and schema 📖

one of graphql's key strengths is that the schema is the **source of truth**.

### the schema defines:

- types
- fields
- relationships
- query and mutation capabilities

### this enables:

- automatic documentation
- strong tooling (autocomplete, validation)
- better developer experience

in rest, documentation is often external (swagger, readme files) and can drift from implementation.

---

## 7. why teams choose graphql 💡

common reasons to use graphql:

- complex, data-driven uis
- multiple clients with different data needs
- rapid iteration on frontend features
- strong need for typed contracts between frontend and backend

graphql is especially effective when frontend teams need autonomy over data shape.

---

## 8. when graphql is a bad idea ⚠️

graphql is not universally better than rest.

it can be a poor choice when:

- apis are simple crud
- data access patterns are stable
- team size is small
- operational simplicity is critical
- caching via http/cdn is a priority

### graphql adds complexity in:

- query execution
- performance tuning
- error handling
- monitoring

---

## 9. when you should NOT use graphql 🚫

avoid graphql when:

- you need simple request/response apis
- you rely heavily on http semantics (status codes, caching, redirects)
- you want easy cdn caching
- you are building file uploads or streaming apis
- you cannot control query complexity

in these cases, rest is often simpler and more predictable.

---

## 10. trade-offs and operational considerations ⚖️

graphql requires careful handling of:

- query complexity and depth
- n+1 query problems
- authorization at field level
- performance monitoring

without proper safeguards, poorly designed queries can degrade backend performance.

---

## 11. what engineers like about graphql 💙

common advantages cited by developers:

- precise data fetching
- strong typing via schema
- excellent tooling
- reduced backend/frontend coupling

the schema acts as a contract that improves collaboration between teams.

---

## 12. summary 🧙‍♀️

graphql is an api paradigm designed to solve data-fetching inefficiencies in complex, client-driven applications. it provides flexibility and strong typing at the cost of increased server-side complexity. rest remains a better choice for simpler, stable apis where operational simplicity and caching are priorities.

---

## key takeaways ✨

- graphql = single endpoint, client-defined queries
- rest = multiple endpoints, server-defined responses
- graphql solves overfetching and underfetching
- graphql schema = automatic documentation
- use graphql for complex, evolving uis
- use rest for simple, stable apis
- graphql adds server complexity
- rest is simpler to cache and monitor
