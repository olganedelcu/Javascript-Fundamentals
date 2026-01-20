# Next.js Fundamentals

## what Next.js is

Next.js is a React-based framework created by Vercel (formerly ZEIT). It was first released in 2016. Unlike React, which is a UI library, Next.js is a full-stack framework that provides routing, rendering strategies, and performance optimizations out of the box.

Next.js uses React as its core rendering layer, but extends it with server-side capabilities and build-time optimizations.

---

## why Next.js was created

React was designed as a client-side UI library. Its responsibility is to describe how the UI changes in response to state and props. It intentionally does not solve problems such as:

- how pages are routed
- how content is rendered before JavaScript loads
- how to handle SEO
- how to optimize initial load performance
- how to handle server-side data fetching

As React applications grew larger and more product-facing, teams had to build custom solutions for these problems using additional tooling and infrastructure.

Next.js was created to standardize and simplify these patterns by providing a framework that solves them in a consistent, production-ready way.

---

## why react alone is not enough in some cases

A typical React application renders entirely in the browser:

1. the browser downloads a mostly empty HTML file
2. JavaScript is loaded
3. React executes and renders the UI

This approach is known as **client-side rendering (CSR)**.

While CSR works well for highly interactive applications, it has drawbacks:

- slow first contentful paint
- poor SEO for content-heavy pages
- blank screens while JavaScript loads
- extra work required for social sharing previews

For applications where performance, discoverability, or content visibility matter, CSR alone is often insufficient.

---

## what problem Next.js solves

Next.js addresses these limitations by enabling multiple rendering strategies:

- **server-side rendering (SSR)**
- **static site generation (SSG)**
- **incremental static regeneration (ISR)**

This allows developers to choose the most appropriate rendering model per page, rather than forcing a single approach for the entire application.

---

## browser vs server responsibilities

### in a traditional React app (CSR)

- rendering happens in the browser
- the server serves static assets only
- SEO relies on JavaScript execution

### in a Next.js app

- the server can render HTML
- data can be fetched on the server
- the browser hydrates the HTML and adds interactivity

This separation allows Next.js to optimize both initial load and interactivity.

---

## why SEO matters

Search engines prioritize:

- fast-loading pages
- immediately available content
- clean, crawlable HTML

Although modern search engines can execute JavaScript, doing so is:

- slower
- less reliable
- deferred compared to HTML parsing

Pages that ship meaningful HTML early are indexed more reliably and rank better.

---

## server-side rendering (SSR)

Server-side rendering means generating HTML for a page on the server for each request.

### flow:

1. request arrives
2. server fetches data
3. React renders HTML
4. HTML is sent to the browser
5. browser hydrates the page

### SSR is useful when:

- data changes frequently
- content must be fresh on every request
- SEO is critical

### trade-offs:

- higher server cost
- slower response compared to static files

### example:

```typescript
// pages/product.tsx
export async function getServerSideProps(context) {
  // runs on the server for every request
  const res = await fetch(`https://api.example.com/product/${context.params.id}`);
  const product = await res.json();

  return {
    props: { product }
  };
}

export default function Product({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```

---

## static site generation (SSG)

Static site generation means generating HTML at build time.

### flow:

1. pages are rendered during build
2. HTML files are stored on a CDN
3. requests are served instantly

### SSG is ideal when:

- content changes infrequently
- pages are content-heavy
- performance is a top priority

Next.js also supports **incremental static regeneration (ISR)**, allowing static pages to be updated after deployment without rebuilding the entire site.

### example:

```typescript
// pages/blog/[slug].tsx
export async function getStaticProps({ params }) {
  // runs at BUILD time
  const res = await fetch(`https://api.example.com/posts/${params.slug}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 3600 // ISR: revalidate every hour
  };
}

export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    paths: posts.map(post => ({
      params: { slug: post.slug }
    })),
    fallback: 'blocking'
  };
}

export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

---

## why google prefers HTML

HTML allows search engines to:

- index content immediately
- avoid executing JavaScript
- understand page structure reliably

Although Google can execute JavaScript, it does so in a secondary rendering pass, which may delay indexing and ranking.

Providing pre-rendered HTML improves:

- crawl efficiency
- ranking stability
- social sharing previews

### what google sees:

**with plain React (CSR):**
```html
<html>
  <body>
    <div id="root"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```

**with Next.js (SSR/SSG):**
```html
<html>
  <body>
    <div id="root">
      <article>
        <h1>How to Use Next.js</h1>
        <p>Full content here...</p>
      </article>
    </div>
    <script src="bundle.js"></script>
  </body>
</html>
```

---

## how Next.js improves performance

Next.js includes several built-in performance optimizations:

### 1. code splitting per page

Each page only loads the JavaScript it needs:

```
pages/
  home.tsx    → only loads home.js
  about.tsx   → only loads about.js
  blog.tsx    → only loads blog.js
```

### 2. automatic static optimization

Next.js automatically determines which pages can be statically generated.

### 3. image optimization

```typescript
import Image from 'next/image';

<Image
  src="/photo.jpg"
  width={800}
  height={600}
  alt="description"
/>
```

**automatic:**
- responsive images
- lazy loading
- modern formats (WebP, AVIF)
- size optimization

### 4. prefetching of linked pages

Links in the viewport are prefetched automatically:

```typescript
import Link from 'next/link';

<Link href="/about">About</Link>
// prefetched when visible
```

### 5. server-side data fetching

Data can be fetched on the server, reducing client-side requests.

By reducing JavaScript sent to the browser and delivering HTML early, Next.js improves perceived and actual performance.

---

## when to use Next.js

### Next.js is a good choice when:

- ✅ SEO is important
- ✅ initial load performance matters
- ✅ pages are content-driven
- ✅ you want server-side data fetching
- ✅ you want a production-ready React framework

### it may be unnecessary when:

- ❌ the app is purely internal
- ❌ SEO does not matter
- ❌ the app is highly interactive with minimal content

---

## rendering strategies comparison

| strategy | when rendered | performance | SEO | use case |
|----------|---------------|-------------|-----|----------|
| **CSR** | browser (client-side) | slower initial load | poor | dashboards, admin panels |
| **SSR** | server (per request) | fast initial load | excellent | dynamic content, personalized |
| **SSG** | build time | fastest | excellent | blogs, marketing, docs |
| **ISR** | build time + periodic rebuild | fastest with fresh data | excellent | e-commerce, news |

---

## client-side rendering (CSR)

### how it works:

```
1. user requests page
   ↓
2. server sends minimal HTML
   ↓
3. browser downloads JavaScript
   ↓
4. JavaScript executes
   ↓
5. React renders content
   ↓
6. user sees page
```

### example:

```typescript
function Dashboard() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/dashboard')
      .then(res => res.json())
      .then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;

  return <div>{data.content}</div>;
}
```

**problems:**
- search engines see "Loading..."
- slow on poor connections
- blank screen during load

---

## comparison: React vs Next.js

| feature | React | Next.js |
|---------|-------|---------|
| **type** | UI library | full-stack framework |
| **rendering** | client-side only | SSR, SSG, ISR, CSR |
| **routing** | requires react-router | file-based (built-in) |
| **SEO** | poor (requires workarounds) | excellent (built-in) |
| **performance** | slower initial load | optimized by default |
| **code splitting** | manual | automatic |
| **data fetching** | client-side only | server-side + client-side |
| **API routes** | need separate backend | built-in |
| **image optimization** | manual | automatic |
| **deployment** | any static host | optimized for Vercel |

---

## hydration explained

Hydration is the process where Next.js attaches React event handlers to server-rendered HTML.

### flow:

1. **server renders HTML** - full content visible
2. **HTML sent to browser** - user sees content
3. **JavaScript loads** - in parallel
4. **React hydrates** - adds interactivity
5. **page becomes interactive** - buttons work, state updates

### why hydration matters:

- users see content before JavaScript loads
- progressive enhancement
- best of both worlds: fast initial load + full interactivity

### example:

```typescript
// this button's HTML is rendered on the server
// but onClick only works after hydration
<button onClick={() => console.log('clicked')}>
  Click me
</button>
```

---

## key mental models

### HTML is king for SEO
- search engines prefer full HTML
- JavaScript is a bonus, not a requirement
- Next.js delivers HTML first

### server = speed for first load
- server can render faster than browser
- user sees content before JavaScript loads
- perception matters

### static = fastest possible
- pre-built HTML
- no server rendering needed
- CDN everywhere

### hybrid is powerful
- mix strategies per page
- optimize each page differently
- best of all worlds

---

## summary

Next.js is a React framework created to solve rendering, performance, and SEO limitations of client-side React applications. By supporting server-side and build-time rendering strategies, it enables faster, more discoverable, and more scalable web applications without requiring custom infrastructure.

### core benefits:

- multiple rendering strategies (SSR, SSG, ISR)
- improved SEO through pre-rendered HTML
- better performance via code splitting and optimization
- production-ready features out of the box
- enhanced developer experience with file-based routing

### when to choose Next.js:

- content-driven applications
- SEO-critical projects
- performance-sensitive sites
- teams wanting a complete framework

### when to stick with React:

- internal tools
- admin panels
- highly interactive apps where SEO doesn't matter
- projects that need minimal framework overhead

---

## summary checklist

- [ ] understand that Next.js is a React framework, not a replacement
- [ ] know the difference between CSR, SSR, SSG, and ISR
- [ ] recognize when SEO matters for your project
- [ ] understand browser vs server responsibilities
- [ ] know why google prefers HTML over JavaScript
- [ ] understand hydration (server HTML → interactive JavaScript)
- [ ] recognize automatic optimizations in Next.js
- [ ] know when to use each rendering strategy
- [ ] understand that static (SSG) is fastest
- [ ] remember: if SEO matters → consider Next.js