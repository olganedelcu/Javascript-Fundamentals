# 🔥 REST fundamentals & integration mindset

## HTTP methods

### the main 5:
| method | purpose | has body? | idempotent? |
|--------|---------|-----------|-------------|
| **GET** | read/fetch data | no | yes |
| **POST** | create new resource | yes | no |
| **PUT** | replace entire resource | yes | yes |
| **PATCH** | update part of a resource | yes | yes |
| **DELETE** | remove a resource | rarely | yes |

### examples:
```
GET    /api/users          → get all users
GET    /api/users/123      → get user 123
POST   /api/users          → create new user
PUT    /api/users/123      → replace user 123 entirely
PATCH  /api/users/123      → update user 123 partially
DELETE /api/users/123      → delete user 123
```

---

## idempotency

an operation is **idempotent** if calling it **multiple times** produces the **same result** as calling it once.

### why it matters:
- network failures can cause duplicate requests
- retries must be safe
- idempotent operations can be safely retried

### which methods are idempotent:
- **GET** — reading the same resource always returns the same thing
- **PUT** — replacing with the same data = same result
- **DELETE** — deleting something already deleted = still gone
- **POST** — **NOT idempotent** — creating twice = two resources

### making POST idempotent:
use an **idempotency key** — a unique identifier sent with the request:
```js
const response = await fetch("/api/payments", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Idempotency-Key": "unique-request-id-12345",
  },
  body: JSON.stringify({ amount: 100, currency: "USD" }),
});
// if this request is sent twice with the same key,
// the server processes it only once
```

---

## pagination

when an API has a lot of data, it returns results in **pages** instead of all at once.

### offset-based pagination (most common)
```
GET /api/users?page=1&limit=20     → users 1-20
GET /api/users?page=2&limit=20     → users 21-40
GET /api/users?offset=0&limit=20   → same as page 1
```

### cursor-based pagination (better for large datasets)
```
GET /api/users?limit=20                        → first 20
GET /api/users?limit=20&cursor=abc123          → next 20 after cursor
```

### typical paginated response:
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "hasMore": true,
    "nextCursor": "abc123"
  }
}
```

### fetching all pages:
```js
async function fetchAllPages(baseUrl) {
  const allData = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const res = await fetch(`${baseUrl}?page=${page}&limit=100`);
    const json = await res.json();
    allData.push(...json.data);
    hasMore = json.pagination.hasMore;
    page++;
  }

  return allData;
}
```

---

## rate limiting

APIs limit how many requests you can make in a given time period.

### common headers:
```
X-RateLimit-Limit: 100          → max requests per window
X-RateLimit-Remaining: 42       → requests left
X-RateLimit-Reset: 1234567890   → when the window resets (unix timestamp)
Retry-After: 30                 → seconds to wait (when rate limited)
```

### handling rate limits:
```js
async function fetchWithRateLimit(url) {
  const res = await fetch(url);

  if (res.status === 429) {
    const retryAfter = res.headers.get("Retry-After") || "5";
    const waitMs = parseInt(retryAfter) * 1000;
    console.log(`rate limited — waiting ${waitMs}ms`);
    await new Promise(resolve => setTimeout(resolve, waitMs));
    return fetchWithRateLimit(url); // retry
  }

  return res.json();
}
```

---

## webhooks

a webhook is when an **external service calls YOUR API** to notify you of an event.

instead of polling (asking "anything new?" every 5 seconds), the service tells you when something happens.

### how it works:
```
1. you register a webhook URL with the service
   → "send events to https://myapp.com/webhooks/stripe"

2. when an event happens (e.g. payment received):
   → stripe sends a POST request to your URL

3. your server processes the event
```

### handling webhooks in express:
```js
app.post("/webhooks/stripe", (req, res) => {
  const event = req.body;

  switch (event.type) {
    case "payment_intent.succeeded":
      console.log("payment received:", event.data);
      break;
    case "customer.created":
      console.log("new customer:", event.data);
      break;
  }

  // always respond with 200 to acknowledge receipt
  res.status(200).json({ received: true });
});
```

### webhook best practices:
- **always return 200 quickly** — process heavy work async
- **verify signatures** — make sure the request is actually from the service
- **handle duplicates** — webhooks can be sent more than once
- **log everything** — for debugging

---

## data mapping / transformation

when integrating with external APIs, the data shape often doesn't match what you need.

### example: transforming API response
```js
// external API returns:
const apiUser = {
  first_name: "Olga",
  last_name: "Nedelcu",
  email_address: "olga@test.com",
  created_at: "2024-01-15T10:00:00Z",
};

// your app expects:
function mapUser(external) {
  return {
    firstName: external.first_name,
    lastName: external.last_name,
    email: external.email_address,
    createdAt: new Date(external.created_at),
    fullName: `${external.first_name} ${external.last_name}`,
  };
}

const user = mapUser(apiUser);
```

### mapping arrays
```js
const apiUsers = await fetchUsers();
const users = apiUsers.map(mapUser);
```

### handling missing/optional fields
```js
function mapUser(external) {
  return {
    name: external.name ?? "unknown",
    email: external.email ?? null,
    age: external.age != null ? Number(external.age) : null,
    roles: external.roles ?? [],
  };
}
```

---

## schema validation

before trusting data from an external source, **validate it**.

### why validate:
- external APIs can change without warning
- data can be malformed, missing fields, wrong types
- prevents runtime crashes deep in your code

### manual validation
```js
function validateUser(data) {
  if (!data || typeof data !== "object") {
    throw new Error("invalid user data");
  }
  if (typeof data.name !== "string") {
    throw new Error("name must be a string");
  }
  if (typeof data.email !== "string" || !data.email.includes("@")) {
    throw new Error("invalid email");
  }
  return data;
}
```

### with zod (popular validation library)
```ts
import { z } from "zod";

const UserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  age: z.number().optional(),
});

// validate
const result = UserSchema.safeParse(apiResponse);

if (result.success) {
  const user = result.data; // fully typed!
} else {
  console.log("validation errors:", result.error.issues);
}
```

zod is extremely common in typescript codebases because it gives you **runtime validation + type inference** in one tool.

---

## summary checklist

- [ ] know all HTTP methods and when to use each
- [ ] understand idempotency and why it matters
- [ ] know how to handle paginated APIs
- [ ] understand rate limiting and how to handle 429 responses
- [ ] know what webhooks are and how to handle them
- [ ] can transform/map data between different API shapes
- [ ] understand why schema validation matters
- [ ] comfortable with auth patterns (bearer tokens, API keys)
