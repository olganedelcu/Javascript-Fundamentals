# 🟢 node.js fundamentals

## 1. what node.js actually is

node.js is a **javascript runtime** built on chrome's V8 engine. it allows you to run javascript **outside the browser** — on servers, scripts, CLI tools, etc.

it is **not** a framework. it is **not** a language. it is a runtime environment.

---

## 2. why node.js was created

before node.js (2009), server-side development was dominated by languages like **PHP, Java, Ruby, Python**. each had their own ecosystems, and frontend developers had to learn a completely different language for the backend.

**ryan dahl** created node.js to solve specific problems he saw in web servers:

### the problem with traditional servers (apache, java):
```
traditional server (thread-per-request model):

request 1 → thread 1 → waits for DB → responds
request 2 → thread 2 → waits for DB → responds
request 3 → thread 3 → waits for DB → responds
...
request 10,000 → ❌ no threads left → connection refused
```
each request spawned a **new thread**. threads are expensive — they consume memory (~2MB each), and most of the time they're just **sitting idle**, waiting for I/O (database queries, file reads, API calls).

### what node.js does differently:
```
node.js (single-threaded event loop):

request 1 → start DB query → move on
request 2 → start DB query → move on
request 3 → start file read → move on
...
request 10,000 → start API call → move on

DB query 1 done → callback runs → respond to request 1
file read done → callback runs → respond to request 3
```
**one thread handles everything** — it never waits. it delegates I/O to the OS and moves to the next task. when the I/O completes, a callback fires.

### why this matters:
- **10,000 concurrent connections** with minimal memory
- **no thread management** overhead
- **same language** on frontend and backend (javascript)
- **npm ecosystem** — largest package registry in the world

---

## 3. what problem node.js solves

node.js is designed for **I/O-bound, event-driven** applications:

- **API servers** — handling thousands of requests that mostly wait for databases/external APIs
- **real-time apps** — chat, live dashboards, collaborative editing (websockets)
- **microservices** — lightweight services that talk to each other
- **integration layers** — unified APIs that aggregate data from multiple sources
- **streaming** — processing data in chunks without loading everything into memory

### what node.js is NOT good for:
- **CPU-intensive tasks** — video encoding, image processing, heavy computation
- **why?** — because while the CPU is busy computing, the event loop is blocked. no other requests can be served

### senior-level insight:
when someone says "node doesn't scale" — they usually mean CPU-bound work. for I/O-bound work (which is 90% of web applications), node scales **extremely well**. the real bottleneck is almost always the database or external service, not node itself.

---

## 4. core runtime concepts

### key characteristics:
- **single-threaded** — one main thread of execution
- **event-driven** — uses an event loop to handle async operations
- **non-blocking I/O** — file reads, network calls, etc. don't block the main thread

### how non-blocking I/O works:
```
1. your code calls fs.readFile("data.txt", callback)
2. node hands the file read to the OS (background)
3. your code keeps running — does NOT wait
4. when the file is ready, the callback is placed in the event queue
5. event loop picks it up and runs the callback
```

### the event loop (simplified):
```
   ┌───────────────────────────┐
┌─>│        timers              │ ← setTimeout, setInterval
│  └───────────┬───────────────┘
│  ┌───────────┴───────────────┐
│  │     pending callbacks      │ ← I/O callbacks
│  └───────────┬───────────────┘
│  ┌───────────┴───────────────┐
│  │        poll                │ ← incoming connections, data, etc.
│  └───────────┬───────────────┘
│  ┌───────────┴───────────────┐
│  │        check               │ ← setImmediate
│  └───────────┬───────────────┘
│  ┌───────────┴───────────────┐
│  │     close callbacks        │ ← socket.on('close')
│  └───────────┬───────────────┘
└──────────────┘
```

### what blocks the event loop (avoid these):
```js
// ❌ synchronous file read in a server
const data = fs.readFileSync("huge-file.json"); // blocks EVERYTHING

// ❌ heavy computation
for (let i = 0; i < 1_000_000_000; i++) { /* ... */ } // blocks EVERYTHING

// ❌ JSON.parse on huge strings
const obj = JSON.parse(giantString); // blocks while parsing

// ✅ use async alternatives
const data = await fs.promises.readFile("huge-file.json");

// ✅ offload CPU work to worker threads
import { Worker } from "worker_threads";
```

### worker threads (for CPU-bound work):
```js
// when you truly need CPU work, use worker threads
import { Worker } from "worker_threads";

const worker = new Worker("./heavy-computation.js", {
  workerData: { input: largeDataSet }
});

worker.on("message", (result) => {
  console.log("computation done:", result);
});

worker.on("error", (err) => {
  console.error("worker failed:", err);
});
```
this keeps the main event loop free while heavy work runs in a separate thread.

### process.nextTick vs setImmediate:
```js
// process.nextTick — runs BEFORE the next event loop phase
process.nextTick(() => console.log("nextTick")); // runs first

// setImmediate — runs in the CHECK phase of event loop
setImmediate(() => console.log("immediate")); // runs after

// practical rule: avoid process.nextTick in application code
// it can starve the event loop if used recursively
```

---

## 5. streams (senior-level concept)

streams let you process data **piece by piece** instead of loading everything into memory.

### why streams matter:
```js
// ❌ loading a 2GB file into memory
const data = await fs.promises.readFile("huge-log.txt", "utf-8");
// → uses 2GB of RAM, app crashes on small servers

// ✅ streaming — processes line by line
import { createReadStream } from "fs";
import { createInterface } from "readline";

const stream = createReadStream("huge-log.txt");
const rl = createInterface({ input: stream });

rl.on("line", (line) => {
  // process one line at a time — uses almost no memory
  if (line.includes("ERROR")) console.log(line);
});
```

### 4 types of streams:
- **readable** — source of data (file read, HTTP request body)
- **writable** — destination for data (file write, HTTP response)
- **transform** — modifies data as it passes through (compression, encryption)
- **duplex** — both readable and writable (TCP socket)

### piping streams:
```js
import { createReadStream, createWriteStream } from "fs";
import { createGzip } from "zlib";

// read file → compress → write compressed file
createReadStream("input.txt")
  .pipe(createGzip())
  .pipe(createWriteStream("input.txt.gz"));
```
this is how node.js can handle files larger than available memory.

---

## 6. file system (fs)

### reading files
```js
import { readFile, readFileSync } from "fs";
import { readFile as readFilePromise } from "fs/promises";

// async (callback-based)
readFile("config.json", "utf-8", (err, data) => {
  if (err) throw err;
  console.log(data);
});

// sync (blocks the thread — avoid in servers)
const data = readFileSync("config.json", "utf-8");

// async with promises (modern, preferred)
const data = await readFilePromise("config.json", "utf-8");
```

### writing files
```js
import { writeFile } from "fs/promises";

await writeFile("output.json", JSON.stringify({ name: "olga" }), "utf-8");
```

### checking if a file exists
```js
import { access } from "fs/promises";

try {
  await access("config.json");
  console.log("file exists");
} catch {
  console.log("file does not exist");
}
```

### when you use fs in integration work:
- reading config files
- writing logs
- processing uploaded files
- reading environment configs

---

## 7. working with APIs (critical for integration work)

### basic fetch
```js
const response = await fetch("https://api.example.com/users");
const data = await response.json();
```

### POST request with auth
```js
const response = await fetch("https://api.example.com/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${token}`,
  },
  body: JSON.stringify({
    name: "olga",
    email: "olga@test.com",
  }),
});

if (!response.ok) {
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}

const data = await response.json();
```

### handling HTTP status codes
```js
async function handleResponse(response) {
  switch (response.status) {
    case 200: return await response.json();           // OK
    case 201: return await response.json();           // created
    case 204: return null;                             // no content
    case 400: throw new Error("bad request");
    case 401: throw new Error("unauthorized");
    case 403: throw new Error("forbidden");
    case 404: throw new Error("not found");
    case 429: throw new Error("rate limited");
    case 500: throw new Error("server error");
    default: throw new Error(`unexpected: ${response.status}`);
  }
}
```

### query parameters
```js
const params = new URLSearchParams({
  page: "1",
  limit: "20",
  search: "olga",
});

const url = `https://api.example.com/users?${params}`;
// https://api.example.com/users?page=1&limit=20&search=olga
```

### common auth patterns
```js
// bearer token (most common for APIs)
headers: { "Authorization": `Bearer ${token}` }

// API key
headers: { "X-API-Key": apiKey }

// basic auth
headers: {
  "Authorization": `Basic ${Buffer.from(`${user}:${pass}`).toString("base64")}`
}
```

---

## 8. express basics

even if you don't use express directly, understanding these patterns is important because most node.js backends follow them.

### creating routes
```js
import express from "express";

const app = express();
app.use(express.json()); // parse JSON bodies

// GET
app.get("/api/users", (req, res) => {
  res.json({ users: [] });
});

// POST
app.post("/api/users", (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ id: "123", name, email });
});

// route parameters
app.get("/api/users/:id", (req, res) => {
  const { id } = req.params;
  res.json({ id, name: "olga" });
});

// query parameters
app.get("/api/search", (req, res) => {
  const { q, page } = req.query;
  res.json({ query: q, page });
});

app.listen(3000, () => console.log("server running on port 3000"));
```

### middleware
middleware functions run **between** the request and the response.
```js
// logging middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next(); // pass to next middleware or route handler
}

app.use(logger); // applies to ALL routes

// auth middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "no token" });

  try {
    const user = verifyToken(token);
    req.user = user; // attach to request
    next();
  } catch {
    res.status(401).json({ error: "invalid token" });
  }
}

// apply to specific routes
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

### error handling middleware
```js
// must have 4 parameters — express recognizes it as error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: "something went wrong" });
});
```

---

## 9. environment variables

### why use them
- keep secrets out of code (API keys, database URLs, tokens)
- configure behavior per environment (dev, staging, production)
- never hardcode sensitive data

### using process.env
```js
const apiKey = process.env.API_KEY;
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
```

### .env files
```
# .env
API_KEY=sk-1234567890
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
PORT=3000
NODE_ENV=development
```

### loading .env files
```js
// with dotenv package
import "dotenv/config";

// or
import dotenv from "dotenv";
dotenv.config();

// now process.env.API_KEY works
```

### important rules:
- **never commit .env files** — add to .gitignore
- **never log secrets** — console.log(process.env) is dangerous
- **use different .env files per environment** (.env.local, .env.production)

---

## 10. debugging (critical for on-site interviews)

### reading stack traces
```
TypeError: Cannot read properties of undefined (reading 'name')
    at getUser (/app/src/users.js:15:23)
    at processRequest (/app/src/handler.js:42:10)
    at /app/src/server.js:8:5
```
read **bottom to top**:
1. started at server.js line 8
2. called handler.js line 42
3. crashed at users.js line 15 — tried to read `.name` on `undefined`

### effective logging
```js
// ❌ useless
console.log(data);

// ✅ useful — tells you what & where
console.log("[getUser] response data:", JSON.stringify(data, null, 2));
console.log("[getUser] user count:", data?.users?.length ?? "no users");
```

### understanding error objects
```js
try {
  await fetch(url);
} catch (error) {
  console.log(error.message);   // human-readable description
  console.log(error.name);      // "TypeError", "ReferenceError", etc.
  console.log(error.stack);     // full stack trace
  console.log(error.cause);     // original error (if re-thrown)
}
```

### common debugging scenarios in integration work:
- **mismatched types** — API returns string, you expect number
- **null/undefined** — optional fields not present in response
- **auth failures** — token expired, wrong header format
- **wrong status code** — expecting 200 but getting 201 or 204
- **malformed JSON** — response is not valid JSON

### debugging checklist:
1. check the **exact error message** and stack trace
2. log the **full API response** (status, headers, body)
3. verify the **request** is correct (url, method, headers, body)
4. check **types** — is the data what you expect?
5. check for **null/undefined** at each step
6. compare with **API documentation**

---

## 11. error handling patterns (senior-level)

### custom error classes
```js
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.name = "AppError";
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404, "NOT_FOUND");
  }
}

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400, "VALIDATION_ERROR");
  }
}

// usage
throw new NotFoundError("User");
// → { message: "User not found", statusCode: 404, code: "NOT_FOUND" }
```

### centralized error handling in express
```js
// all routes throw errors — one handler catches them all
app.get("/api/users/:id", async (req, res, next) => {
  try {
    const user = await db.findUser(req.params.id);
    if (!user) throw new NotFoundError("User");
    res.json(user);
  } catch (err) {
    next(err); // pass to error handler
  }
});

// centralized error handler
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const code = err.code || "INTERNAL_ERROR";

  // log for debugging (never expose internals to client)
  console.error(`[${code}] ${err.message}`, err.stack);

  res.status(statusCode).json({
    error: {
      code,
      message: statusCode === 500 ? "internal server error" : err.message,
    },
  });
});
```

### unhandled rejections & uncaught exceptions
```js
// catch promises that nobody caught
process.on("unhandledRejection", (reason, promise) => {
  console.error("unhandled rejection:", reason);
  // log it, alert monitoring, gracefully shut down
});

// catch sync exceptions that nobody caught
process.on("uncaughtException", (error) => {
  console.error("uncaught exception:", error);
  process.exit(1); // exit — the process is in an unknown state
});
```

### graceful shutdown
```js
// when the process receives SIGTERM (e.g. docker stop, deploy)
process.on("SIGTERM", async () => {
  console.log("shutting down gracefully...");

  // stop accepting new connections
  server.close();

  // close database connections
  await db.disconnect();

  // close any other resources
  process.exit(0);
});
```
this prevents dropped requests and data corruption during deployments.

---

## 12. when to use node.js (decision framework)

### node.js is excellent for:
- ✅ REST APIs / GraphQL APIs
- ✅ real-time applications (websockets, chat)
- ✅ microservices and integration layers
- ✅ serverless functions (AWS Lambda, Vercel)
- ✅ CLI tools and build scripts
- ✅ streaming data (file processing, ETL pipelines)

### consider alternatives when:
- ❌ heavy computation (use Go, Rust, or Python with C extensions)
- ❌ strict type safety requirements at runtime (use Java, C#)
- ❌ memory-constrained environments (node's V8 is memory-hungry)

### senior-level perspective:
node.js is not about being "the best language" — it's about **developer velocity** and **ecosystem**. when your team already knows javascript, when you need fast iteration, when your workload is I/O-bound — node is hard to beat. the npm ecosystem means you rarely build from scratch.

the real skill is knowing **when NOT to use it** — and having the maturity to reach for the right tool.

---

## summary checklist

- [ ] understand node.js is a runtime, not a framework
- [ ] know WHY node.js was created (thread-per-request vs event loop)
- [ ] know what problems node.js solves (I/O-bound, real-time, integrations)
- [ ] know single-threaded + event loop model
- [ ] know non-blocking I/O and why it matters
- [ ] understand when the event loop gets blocked and how to avoid it
- [ ] know about worker threads for CPU-bound work
- [ ] understand streams for processing large data
- [ ] comfortable reading/writing files with fs/promises
- [ ] can make API calls with fetch (GET, POST, headers, auth)
- [ ] understand HTTP status codes
- [ ] know express basics (routes, middleware, error handling)
- [ ] use environment variables properly
- [ ] can read and debug stack traces
- [ ] effective logging for debugging
- [ ] know custom error classes and centralized error handling
- [ ] understand graceful shutdown and process signals
- [ ] know when to use node.js and when NOT to
