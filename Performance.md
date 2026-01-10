✅ Cookies vs Local Storage vs Session Storage — Deep Dive

🍪 **Cookies Storage**
Cookies have a strict limit in terms of storage — around **4KB**, which makes them suitable for small but critical pieces of data. One of the most powerful aspects of cookies is that they operate on both the **server and client side**. Every time you send an HTTP request (like visiting a route or fetching data), the browser includes the cookies automatically in the request headers. This behavior makes them perfect for **authentication flows**, particularly when you're working over HTTPS and using security flags like `HttpOnly`, `Secure`, and `SameSite`. This way, the token stored in the cookie is not accessible from JavaScript and is protected from XSS attacks.

Cookies are ideal for storing things like tokens, session identifiers, or anything that needs to be **shared between the client and server seamlessly**, without manually attaching it to every request. Because the browser handles this, they are extremely effective when you want **automatic synchronization** between client-server interactions.

**Key traits:**
- Auto-sent on every HTTP request (GET, POST, etc.)
- Lifespan can be explicitly defined using an expiration date
- Operates across tabs and sessions
- Stored in the browser under the application domain
- Limited size (~4KB)

🗂️ **Local Storage**
LocalStorage, on the other hand, has a **much higher storage limit** (usually around **5–10MB**, depending on the browser). Unlike cookies, it is purely **client-side** and not sent with HTTP requests. It persists even after closing the browser, so it is ideal for storing **application state**, **user preferences**, or **temporary data** that you want to keep across sessions.

For instance, imagine your app has a user onboarding flow and you want to store which step the user is on. If they close the browser and return later, LocalStorage allows you to retrieve that information and **resume exactly where they left off**.

**Key traits:**
- Not sent with HTTP requests (not for auth)
- Lifespan: persists indefinitely unless cleared
- Scope: available to all tabs/windows of the same origin
- Larger storage limit (~5–10MB)

🕒 **Session Storage**
SessionStorage is very similar to LocalStorage in terms of API (both use `setItem`, `getItem`, etc.), but it behaves differently: it is **cleared when the tab or browser window is closed**. It's **scoped per tab**, so if you open the same page in two tabs, each one will have a separate session storage.

SessionStorage is **not used as much**, but it's perfect for use cases like **checkout pages** or **form inputs**, where you don’t want the user to lose progress if the page reloads or something fails, but you also **don’t want to persist that data forever**. Since it’s automatically cleared, you don’t need to implement cleanup logic.

**Key traits:**
- Cleared when tab/window is closed
- Not shared across tabs
- Only available to scripts on the same page/session
- Great for sensitive or temporary data (e.g. form steps, OTPs)

---

Q2: 🧠 **If you are setting up a new frontend application, what are some optimizations you would put in place to make it more performant?**

👷‍♀️ Let's assume this is a **modern frontend React application** using **client-side rendering** and something like **Webpack** or **Vite** as a module bundler.

### 1. Polyfilling the Code (Backward Compatibility)
In any frontend project, you’ll likely want to use **modern JavaScript features** (like async/await, spread operator, optional chaining, etc). But not all browsers support these features — especially older ones like **Internet Explorer 11**. So you need to **polyfill** your code.

💡 **What is a polyfill?** It’s a piece of code (usually a function or method) that provides modern functionality on older browsers that do not natively support it.

Here's how the process works:
- You define your **target browsers** (in `browserslist`)
- You analyze: does this browser support feature X?
- If not → **replace** or **add** a version of that feature that works in older JS.

Example: If the browser doesn’t support `Promise`, you might import a Promise polyfill like `core-js`.

📦 The tradeoff: yes, this **increases the bundle size**, but ensures the app runs on more devices, increasing accessibility.

```js
import 'core-js/features/promise';
import 'whatwg-fetch';
```

---

### 2. Bundle Compression (Gzip/Brotli)
Once your JavaScript is bundled together, you want to reduce the file size as much as possible before it’s sent over the network. Instead of sending a raw `.js` file, you send a **compressed gzip or brotli file**.

This can reduce file size by **up to 70%**, which has a **huge impact on load time**, especially for mobile users or slow networks.

You use **content negotiation** (via the HTTP headers) to tell the browser: "hey, this is a gzip file", and then it automatically decompresses it and executes it like normal.

In Webpack:
```js
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  plugins: [
    new CompressionPlugin({
      algorithm: 'gzip',
    }),
  ],
};
```

---

### 3. Uglification and Minification
This is a classic optimization. You take human-readable JS and remove all **unnecessary characters**:
- newlines
- whitespace
- long variable names

```js
const userLoggedIn = true;
```
gets transformed into:
```js
let a=!0;
```

This process makes your file smaller, but unreadable — this is why you also generate **source maps**, so you can still debug your original code when an error happens in production.

Without source maps:
- You get an error on line 1 of `main.min.js` — useless.

With source maps:
- You know the original line of code in `auth.js:45` that caused it.

---

### 4. Code Splitting (Load What You Need)
Instead of shipping **all** your JavaScript upfront, split your code into logical chunks and load them **on-demand**.

Example:
```tsx
const CheckoutPage = React.lazy(() => import('./Checkout'));
```

This is useful because on initial page load, you might not need the checkout page at all. So don’t load it. Instead, lazy-load it when the user navigates to it.

Webpack/Vite will handle the chunking for you.

---

### 5. Tree Shaking (Eliminate Dead Code)
If you're using **ES6 static imports**, Webpack or Vite can analyze your code and remove unused parts of a library — known as **tree shaking**. 

Example:
```js
import { Button } from 'my-ui-library';
```
If you don’t use `Modal`, `Tooltip`, etc. — those won’t be bundled so performance is as good as possible.

💡 Does not work well with `require()` or dynamic imports.

---

### 6. Dependency Graph
Webpack builds a **dependency graph** by analyzing your `import` statements starting from the `entry` file (usually `index.js`). It forms a **tree structure** of modules and figures out what is needed, what can be bundled together, and what can be eliminated.

This ensures optimal bundling and helps during code splitting and tree shaking.

### ⚙️ 6. Image & Asset Optimization
- I always optimize images by:
- Resizing to only what's needed
- Using **WebP** over PNG/JPEG
- Stripping metadata
- Compressing with tools like `imagemin`
- Hosting on **CDN** with caching headers
- Enable **lazy loading** (`loading="lazy"`) for offscreen images.
- Use `srcSet` for responsive images — smaller images for mobile, larger for desktop.
- Always define `width` and `height` to prevent layout shifts.

---