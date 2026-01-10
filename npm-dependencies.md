# 📦 npm & dependencies
there are 2 files that show up in almost every JavaScript project:
```pgsql
package.json
package-lock.json
```
and then there is also a bunch of stuff under "dependences" and "devDependences"
### 📄 package.json
this file tells us the key things about the project:
- name
- which packages it needs
- which scripts you can run(like **npm run dev**)

You'll usually see two important sections:
###### ✅ **dependences**
these are the packages your project needs to run. 
If you are using React, React will be listed here:
```json
"dependencies": {
  "react": "^18.2.0",
  "axios": "^1.5.0"
}
```
Installed with:
```bash
npm install [package-name]
```

### 🛠 "devDependencies"

These are tools you only need while you're coding — like linters, bundlers, formatters.
```json
"devDependencies": {
  "eslint": "^8.54.0",
  "vite": "^5.1.0"
}
```

Installed with:
```bash
npm install -D [package-name]
```
### 🔒 package-lock.json — the version freeze

this file is created automatically when you install packages.

ot locks in the exact versions of all your dependencies (and their dependencies too).

why does that matter?

because different versions can behave differently. So when you share your project with someone else, this file makes sure everyone installs the exact same versions. No surprises.

yes, it can get big. yes, you should commit it.

### 🛠 Common npm commands

| command | what it does |
|---------|--------------|
| `npm install` | installs everything from package.json |
| `npm install [pkg]` | installs a specific package |
| `npm install -D [pkg]` | installs as a dev dependency |
| `npm run dev` | starts your dev server (like Vite or Next) |
| `npm run build` | builds your app for production |
| `npm run lint` | runs a linter (if set up) |
| `npm outdated` | shows outdated packages |
| `npm audit` | checks for security issues |
### 💬 Why do we need dependencies anyway?

because we don’t want to build everything from scratch.
imagine writing your own router. or state manager. or HTTP client. 😅

dependencies are tools that make development faster and easier.

- want routing? → react-router-dom
- need animations? → framer-motion
- want type checking? → typescript
- fetch data? → axios or fetch

they save us time. And honestly, they make projects a lot more fun to build.

##### Bonus: what happens when you npm run build
when you run this command, npm looks into the scripts section of package.json and executes whatever’s written there.
```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "lint": "eslint ."
}
```

so **npm run build** usually means:
bundle everything, optimize it, minify the code, and get it ready for production.

#### 🧠 tl;dr

- **package.json** — tells npm what your project needs + what scripts it can run
- **package-lock.json** — makes sure everyone installs the exact same versions
- **dependencies** = required for the app to run
- **devDependencies** = required for you to build, test, or lint
- **npm run** = how you trigger custom scripts
