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

---

### CSS and JS is?
used together for building frontend.
CSS handles styles, JS handles interactivity.

#### use cases.
- css-in-js = dynamic styles.
c- ss styles when certain state changes (like when clicking a button).
- you could do it with classes, but it's faster and more powerful with css-in-js.
- you write css inside js files, and then tools like webpack inject them into the html DOM at runtime.
- this lets you do dynamic styling — styles based on props, themes, component state.

#### disadvantages.
- because styles are in js files,
you can’t extract them into static css files.
- so they are not cacheable.
- the browser can’t store them between visits.
- user has to re-download them every time.
also makes styles harder to debug,
- because they’re injected dynamically, class names are hashed,
and don’t show clear line numbers like regular .css files.


### ⚙️ 6. Image & Asset Optimization
##### Frontend apps with very huge images, how would we optimize for performance?
- Dimensions matter: 
- Resizing to only what's needed
- Using **WebP** over PNG/JPEG
- Stripping metadata: once we have the dimensions, we look into compression and image optimization toools that can remove metadata from the image or modify what they call the color space to include less colors. The image looks a bit less vibrant but it contains less data so its smaller.
- Hosting on **CDN** with caching headers and have direct caching polices.
- Enable **lazy loading** (`loading="lazy"`) for offscreen images, load on scroll - load images as user scrolls.
- Use `srcSet` attribute to ship different images depending on the viewport on devide. Modern CDNs makes it by default.
- Specify `width` and `height` to pnot have accumulative layer shift.

---

## Performance challenges in frontend 
### How do you manage code quality in a large scale frontend application? What tools and practices do you use?

🧹 for code quality, i would probably start with a **linter**, so you want to basically catch small issues and make sure everybody writes the same code.
🎨 you’re gonna have **prettier**, maybe **eslint** set up, and if you're using typescript, have the **tslint** set up.
🧠 this takes off a lot of work and communication because everybody writes code in the same style.

🧪 after that, you do want to have a layer of unit tests, and ideally some **E2E** tests for sure.
🔍 and finally, i would have something like a dependency scan.

-  in the linter, have something that scans for **a11y**, which stands for accessibility.

🧰 so now we’re taking care of **code quality**, **style**, **accessibility**, **testing**, and **dependencies** (since node modules especially can be vulnerable to different attacks).

📊 finally, have something like **lighthouse** or **sentry** in your pipeline —
📈 they tell you how your **core web vitals** change and how web performance changes over time.
⚠️ so if we make any mistake, like adding a big image or extra fonts, we **immediately see the effect** and fix it early on.

### ❓ what is an xss attack and how do you make sure that frontend apps are not vulnerable to those attacks?

💥 xss = cross-site scripting.
👾 the attacker persists some javascript code in our database.
🌐 then users, because they fetch from our database, end up running that code in their browser.

### ❓ what are micro-frontends? when would you use frontend architecture?
🧩 in a frontend, we usually have different components — header, body of the page, checkout info, etc.
👥 this works pretty fine when we’re in small teams, but as we **scale**, it becomes really hard to contribute to a **single frontend monolith**.

⚙️ so what we can do is split those individual parts into different applications.
🧱 and then we have a **shell** — kind of like a container — that puts them all together.
🔐 this shell can be responsible for things like **authentication**, and **shared state**.

🚀 now we can deploy the **different apps independently**,
so it allows us to **separate** development teams,
and make development much faster and more modular.

⚠️ BUT you pay t**he price of complexity** —
you need more complex tooling to make something like this happen (routing, shared packages, auth sync, etc).

#### 🧠 when does it make sense?
🧱 when we already have a big monolithic app,
and we’re thinking of **breaking it into micro-frontends**.

👥 micro-frontends are mostly good when **splitting** organisations.
you need to split people into teams that can work **independently**.

🔗 but because we’re distributing our system,
you always pay the price for that — especially things like:

🤯 sharing state
🧪 integration testing
🔀 coordination between teams
✅ so when we need technology to **enable parallel work without cross-dependencies**, micro-frontends can help.

🚫 but if you’re building smaller websites or simple apps, there’s no real reason to overcomplicate —
just stick with a regular monolithic frontend setup.