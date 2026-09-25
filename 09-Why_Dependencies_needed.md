

## 1. What actually happens when GitHub Actions starts?

Your workflow runs on:

```yaml
runs-on: ubuntu-latest
```

GitHub creates a **fresh temporary Ubuntu virtual machine** for this job.

Think of it like:

```text
GitHub Actions
      │
      ▼
Fresh Ubuntu machine
      │
      │ initially:
      │
      ├── Your React project ❌
      ├── node_modules ❌
      ├── Node.js ❌
      └── npm ❌
```

Then your steps prepare this machine.

---

# 2. What does `actions/checkout@v4` do?

You have:

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

This means:

> "Take the code from my GitHub repository and download/checkout it into this GitHub Actions runner."

Before checkout:

```text
GitHub Repository
       │
       │
       ▼
     ☁️ GitHub
       
Runner
       │
       └── Empty
```

After checkout:

```text
GitHub Repository
       │
       │ checkout
       ▼
GitHub Actions Runner
       │
       └── Your repository
            │
            ├── .github/
            ├── 05-using-Pre-Built-Actions/
            │    └── react-app/
            │         ├── package.json
            │         ├── package-lock.json
            │         ├── src/
            │         └── public/
            └── ...
```

### Important:

`checkout` gives you **files**.

It does NOT give you:

```text
node_modules/
```

because `node_modules` is normally not committed to Git.

---

# 3. Why don't we commit `node_modules`?

Suppose your React application has:

```text
package.json
package-lock.json
```

Your `package.json` might contain:

```json
{
  "dependencies": {
    "react": "...",
    "react-dom": "...",
    "axios": "..."
  }
}
```

These files tell npm:

> "These are the dependencies my application needs."

But the actual packages are stored in:

```text
node_modules/
```

You normally have:

```text
react-app/
├── package.json
├── package-lock.json
├── src/
└── public/
```

and **not**:

```text
react-app/
└── node_modules/   ❌
```

because `node_modules` can be huge.

So after checkout, the runner has:

```text
react-app/
├── package.json       ✅
├── package-lock.json  ✅
├── src/               ✅
└── node_modules/      ❌
```

---

# 4. Then what does `setup-node` do?

You have:

```yaml
- name: Setup Node
  uses: actions/setup-node@v3
  with:
    node-version: '20.x'
```

This prepares **Node.js** on the runner.

Before:

```text
Runner

Node.js ❌
npm     ❌
```

After:

```text
Runner

Node.js 20.x ✅
npm        ✅
```

So now the machine knows how to execute:

```bash
node
npm
```

But your React dependencies still aren't installed.

We have:

```text
Node.js       ✅
npm           ✅
React project  ✅
Dependencies  ❌
```

That's where `npm ci` comes in.

---

# 5. What is `npm ci`?

This is probably the most important part.

`npm ci` means:

> **Clean Install**

It installs the dependencies described by your `package-lock.json`.

You have:

```yaml
- name: Install Dependencies
  run: npm ci
```

So npm looks at:

```text
package.json
package-lock.json
```

and downloads the required packages.

For example:

```text
package.json
      +
package-lock.json
      │
      ▼
    npm ci
      │
      ▼
node_modules/
      │
      ├── react
      ├── react-dom
      ├── react-scripts
      ├── typescript
      └── ...
```

Now the application can actually run.

---

# 6. Why `npm ci` instead of `npm install`?

You could use:

```bash
npm install
```

But in CI/CD pipelines, `npm ci` is commonly preferred.

### `npm install`

Generally:

```text
package.json
     ↓
npm install
     ↓
Install dependencies
```

It can also update the lockfile when appropriate.

### `npm ci`

```text
package-lock.json
       ↓
    npm ci
       ↓
Install EXACT locked dependency versions
```

`ci` is designed for **clean, reproducible installations**.

That's why CI pipelines commonly use:

```yaml
run: npm ci
```

instead of:

```yaml
run: npm install
```

---

# 7. Now your biggest question: Why `working-directory`?

Your repository isn't directly a React application.

Your structure is something like:

```text
Git-hub-Actions-Practice/
│
├── 01-...
├── 02-...
├── 03-...
├── 04-...
└── 05-using-Pre-Built-Actions/
     │
     └── react-app/
          │
          ├── package.json
          ├── package-lock.json
          ├── src/
          └── public/
```

GitHub Actions starts commands from the **repository root**.

So when you write:

```yaml
run: npm ci
```

GitHub essentially executes:

```bash
cd Git-hub-Actions-Practice
npm ci
```

But your `package.json` is actually here:

```text
Git-hub-Actions-Practice/
└── 05-using-Pre-Built-Actions/
    └── react-app/
        └── package.json
```

Therefore npm would say something like:

```text
npm error
Could not find package.json
```

because you're in the wrong directory.

---

# 8. That's why you use `working-directory`

You wrote:

```yaml
working-directory: 05-using-Pre-Built-Actions/react-app
```

This tells GitHub:

> "Before executing this particular command, move into this directory."

So:

```yaml
- name: Install Dependencies
  run: npm ci
  working-directory: 05-using-Pre-Built-Actions/react-app
```

is effectively:

```bash
cd 05-using-Pre-Built-Actions/react-app
npm ci
```

Then npm finds:

```text
package.json        ✅
package-lock.json   ✅
```

and installs the dependencies.

---

# 9. Why do you need it again for `npm run test`?

You have:

```yaml
- name: Run Unit Tests
  run: npm run test
  working-directory: 05-using-Pre-Built-Actions/react-app
```

Because each `run` step executes a command, and you need that command to execute in the React application's directory.

Conceptually:

```text
Install Dependencies

cd react-app
npm ci
```

Then:

```text
Run Tests

cd react-app
npm run test
```

The second command needs to know **which `package.json` contains the `test` script**.

For example:

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test"
  }
}
```

So:

```bash
npm run test
```

needs to be executed where that `package.json` exists.

---

# 10. Your complete workflow

Your workflow is basically doing this:

```text
                 GitHub
                   │
                   │ checkout
                   ▼
        ┌─────────────────────┐
        │ GitHub Actions      │
        │ Ubuntu Runner       │
        └─────────────────────┘
                   │
                   ▼
          Repository files
                   │
                   ▼
             Setup Node 20
                   │
                   ▼
             Node.js + npm
                   │
                   ▼
             npm ci
                   │
                   ▼
          node_modules/
                   │
                   ▼
          npm run test
                   │
                   ▼
             Test results
```

So each action has a different responsibility:

| Step           | Purpose                      |
| -------------- | ---------------------------- |
| `checkout`     | Get your source code         |
| `setup-node`   | Install/configure Node.js    |
| `npm ci`       | Install project dependencies |
| `npm run test` | Execute your tests           |

---

# 11. A very important distinction

Think about your **local computer**.

You probably already did:

```powershell
npm install
```

or:

```powershell
npm ci
```

So your computer already has:

```text
node_modules/
```

Then you run:

```powershell
npm test
```

and it works.

But GitHub Actions doesn't use your computer.

It creates a **new runner**.

Your local machine:

```text
Your PC
│
├── Node.js ✅
├── npm ✅
├── project ✅
└── node_modules ✅
```

GitHub Actions:

```text
Fresh Runner
│
├── Node.js ❌ → setup-node
├── npm ❌     → setup-node
├── project ❌ → checkout
└── node_modules ❌ → npm ci
```

That's the whole reason these steps exist.

---

# 12. One more thing: `working-directory` doesn't have to be repeated

You **can** avoid repeating it.

You can set a default working directory for all `run` commands:

```yaml
defaults:
  run:
    working-directory: 05-using-Pre-Built-Actions/react-app
```

Then:

```yaml
name: 05 - using Actions

on:
  push:

defaults:
  run:
    working-directory: 05-using-Pre-Built-Actions/react-app

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '20.x'

      - name: Install Dependencies
        run: npm ci

      - name: Run Unit Tests
        run: npm run test
```

Now every `run:` command automatically runs inside:

```text
05-using-Pre-Built-Actions/react-app
```

Notice that `checkout` doesn't use the working directory because `checkout` is an **action**, not a shell `run` command.

---

## The simplest way to remember

Think of your GitHub Actions pipeline as preparing a new computer:

```text
1. checkout
   ↓
"Give me my project."

2. setup-node
   ↓
"Give me Node.js."

3. npm ci
   ↓
"Give me all the libraries my project needs."

4. npm run test
   ↓
"Now test my project."
```

And:

```yaml
working-directory:
```

means:

> **"Run this command from this folder."**

So **checkout does not mean install**.

**Checkout = source code**

**npm ci = dependencies**

**setup-node = Node.js environment**

**test = actually execute your application tests**.
