In **GitHub Actions**, an **event filter** lets you control **exactly when a workflow should run**.

For example, instead of saying:

> "Run this workflow on every push."

you can say:

> "Run this workflow only when code is pushed to the `main` branch."

or:

> "Run it only when files inside the `frontend/` folder change."

---

# 1. What is an Event Filter?

First understand the difference:

### Event

An **event** is something that happens in GitHub.

Examples:

```yaml
on:
  push:
```

This means:

> Run the workflow when someone pushes code.

Other events:

```yaml
on:
  pull_request:
```

Run when a Pull Request is created/updated.

```yaml
on:
  issues:
```

Run when an issue changes.

---

### Filter

A **filter** adds conditions to that event.

For example:

```yaml
on:
  push:
    branches:
      - main
```

Meaning:

> The event is `push`, but only pushes to `main` should trigger the workflow.

So:

```text
Event
  ↓
push
  ↓
Filter
  ↓
Which branch?
  ↓
main
  ↓
Run workflow
```

---

# 2. The Main Filters

For `push` and `pull_request`, the important filters are:

| Filter            | What it checks                 |
| ----------------- | ------------------------------ |
| `branches`        | Which branches                 |
| `branches-ignore` | Which branches to exclude      |
| `tags`            | Which tags                     |
| `tags-ignore`     | Which tags to exclude          |
| `paths`           | Which files/folders changed    |
| `paths-ignore`    | Which files/folders to exclude |

Let's understand each one.

---

# 3. `branches`

`branches` specifies **which branches can trigger the workflow**.

Example:

```yaml
name: My Workflow

on:
  push:
    branches:
      - main
```

This means:

```text
Push to main       → ✅ Workflow runs
Push to develop    → ❌ Workflow doesn't run
Push to feature-x  → ❌ Workflow doesn't run
```

### Real example

Suppose your repository has:

```text
main
develop
feature/login
feature/payment
```

You want your CI/CD pipeline to run only when code reaches `main`.

```yaml
on:
  push:
    branches:
      - main
```

Now:

```text
git push origin main
        ↓
Workflow runs
```

But:

```text
git push origin develop
        ↓
Workflow doesn't run
```

---

# 4. Multiple branches

You can specify multiple branches.

```yaml
on:
  push:
    branches:
      - main
      - develop
```

Now:

```text
main       → ✅
develop    → ✅
feature/*  → ❌
```

---

# 5. Branch patterns

You can use patterns.

```yaml
on:
  push:
    branches:
      - 'feature/**'
```

This means:

> Run when pushing to branches under `feature/`.

For example:

```text
feature/login       → ✅
feature/payment     → ✅
feature/user/profile → ✅
develop             → ❌
main                → ❌
```

Another example:

```yaml
branches:
  - 'release/**'
```

Matches:

```text
release/v1
release/v2
release/production
```

---

# 6. `branches-ignore`

`branches-ignore` means:

> Run for branches **except** these branches.

Example:

```yaml
on:
  push:
    branches-ignore:
      - develop
```

Meaning:

```text
main       → ✅
feature-x  → ✅
develop    → ❌
```

So:

```text
branches
    ↓
"I want these branches"

branches-ignore
    ↓
"I don't want these branches"
```

### Example

Suppose you want the workflow to run everywhere except `main`:

```yaml
on:
  push:
    branches-ignore:
      - main
```

Then:

```text
main       → ❌
develop    → ✅
feature-x  → ✅
```

---

# 7. `branches` vs `branches-ignore`

This is very important.

### `branches`

You specify what **you want**.

```yaml
branches:
  - main
  - develop
```

Meaning:

> Only `main` and `develop`.

---

### `branches-ignore`

You specify what **you don't want**.

```yaml
branches-ignore:
  - develop
```

Meaning:

> Everything except `develop`.

### Simple memory trick

```text
branches
    = INCLUDE these branches

branches-ignore
    = EXCLUDE these branches
```

---

# 8. `tags`

A **Git tag** is a name attached to a specific commit, commonly used for releases.

For example:

```text
v1.0.0
v1.1.0
v2.0.0
```

Suppose you want your deployment workflow to run only when a release tag is pushed.

```yaml
on:
  push:
    tags:
      - 'v*'
```

Now:

```text
v1.0.0     → ✅
v1.2.0     → ✅
v2.0.0     → ✅

test       → ❌
release1   → ❌
```

The `*` is a wildcard.

---

# 9. Real-world tag example

Imagine your project follows:

```text
v1.0.0
v1.1.0
v1.2.0
v2.0.0
```

You could have:

```yaml
name: Production Deployment

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: echo "Deploying application..."
```

When you do:

```bash
git tag v1.0.0
git push origin v1.0.0
```

GitHub sees:

```text
New tag pushed
       ↓
v1.0.0
       ↓
Matches v*
       ↓
Workflow runs
```

This is commonly useful for **release/deployment workflows**.

---

# 10. `tags-ignore`

`tags-ignore` means:

> Run for tags except the specified tags.

Example:

```yaml
on:
  push:
    tags-ignore:
      - 'beta-*'
```

Now:

```text
v1.0.0       → ✅
v2.0.0       → ✅
beta-1.0     → ❌
beta-2.0     → ❌
```

---

# 11. `tags` vs `tags-ignore`

Same concept as branches.

```yaml
tags:
  - 'v*'
```

means:

> Include these tags.

Whereas:

```yaml
tags-ignore:
  - 'beta-*'
```

means:

> Exclude these tags.

---

# 12. `paths`

Now this is very important for real projects.

`paths` checks **which files were changed**.

Suppose your project looks like:

```text
my-project/
│
├── frontend/
│   ├── index.html
│   ├── app.js
│   └── style.css
│
├── backend/
│   ├── app.py
│   └── requirements.txt
│
└── README.md
```

Suppose you have a workflow specifically for the frontend.

```yaml
on:
  push:
    paths:
      - 'frontend/**'
```

This means:

> Run the workflow only when something inside `frontend/` changes.

---

### Example 1

You modify:

```text
frontend/app.js
```

Result:

```text
frontend/app.js changed
        ↓
Matches frontend/**
        ↓
✅ Workflow runs
```

---

### Example 2

You modify:

```text
backend/app.py
```

Result:

```text
backend/app.py changed
        ↓
Doesn't match frontend/**
        ↓
❌ Workflow doesn't run
```

---

# 13. Why `paths` is useful

Imagine you have a large project:

```text
project/
│
├── frontend/
├── backend/
├── mobile/
├── documentation/
└── infrastructure/
```

You might have separate workflows:

```text
frontend-ci.yml
backend-ci.yml
mobile-ci.yml
terraform.yml
```

You don't want the frontend workflow to run when only Terraform files change.

So:

```yaml
# frontend-ci.yml

on:
  push:
    paths:
      - 'frontend/**'
```

And:

```yaml
# backend-ci.yml

on:
  push:
    paths:
      - 'backend/**'
```

This makes your CI more efficient.

---

# 14. `paths-ignore`

`paths-ignore` means:

> Run the workflow except when only these files change.

Example:

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
```

Meaning:

> Don't trigger the workflow for changes only inside `docs/`.

For example:

```text
docs/README.md
        ↓
❌ Workflow doesn't run
```

But:

```text
src/app.js
        ↓
✅ Workflow runs
```

---

# 15. Very important: "only" with `paths-ignore`

Consider:

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
```

Suppose the commit changes:

```text
docs/README.md
src/app.js
```

Because `src/app.js` is **not ignored**, the workflow can run.

So `paths-ignore` is essentially useful when you want to say:

> "Don't run if the changes are only in these paths."

---

# 16. `paths` vs `paths-ignore`

Again, remember:

### `paths`

```yaml
paths:
  - 'frontend/**'
```

Means:

> Run only when these paths change.

### `paths-ignore`

```yaml
paths-ignore:
  - 'docs/**'
```

Means:

> Don't run when changes are only in these paths.

---

# 17. Combining Branch + Path

This is where GitHub Actions becomes powerful.

You can combine filters.

Example:

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'backend/**'
```

This means:

> Run only when a push happens to `main` AND changes something inside `backend/`.

Think:

```text
              Push
                ↓
           Is branch main?
             /      \
           NO        YES
           ↓          ↓
          ❌      Backend changed?
                       /    \
                     NO      YES
                     ↓        ↓
                    ❌       ✅
```

---

# 18. Real Corporate Example

Imagine BMW-SPAREHUB:

```text
BMW-SPAREHUB/
│
├── frontend/
│   ├── src/
│   └── package.json
│
├── backend/
│   ├── src/
│   └── pom.xml
│
├── terraform/
│   └── main.tf
│
└── README.md
```

You might have:

### Frontend CI

```yaml
name: Frontend CI

on:
  push:
    branches:
      - main
    paths:
      - 'frontend/**'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: npm ci
        working-directory: frontend

      - run: npm run build
        working-directory: frontend
```

This workflow runs when:

```text
Branch = main
AND
frontend/** changed
```

---

# 19. Backend CI

```yaml
name: Backend CI

on:
  push:
    branches:
      - main
    paths:
      - 'backend/**'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: ./mvnw test
        working-directory: backend
```

Now:

```text
Change frontend
       ↓
Frontend CI → ✅
Backend CI   → ❌
```

And:

```text
Change backend
       ↓
Frontend CI → ❌
Backend CI   → ✅
```

---

# 20. Combining branches and tags

You can also configure branch and tag triggers.

For example:

```yaml
on:
  push:
    branches:
      - main
    tags:
      - 'v*'
```

This can trigger for:

```text
push to main
    → workflow runs

push tag v1.0.0
    → workflow runs
```

But note an important GitHub Actions rule: when configuring `branches`/`branches-ignore` or `tags`/`tags-ignore`, the branch/tag filter behavior matters for the event, and you should structure the trigger based on whether you want branch pushes, tag pushes, or both.

---

# 21. Combining `branches` + `paths`

This is one of the most useful combinations.

```yaml
on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'
```

Meaning:

```text
             PUSH
               ↓
       ┌───────┴────────┐
       ↓                ↓
 main/develop?       other branch
       ↓                ↓
      YES              ❌
       ↓
 src/** changed?
    /       \
  YES       NO
   ↓         ↓
  ✅        ❌
```

---

# 22. Multiple paths

You can specify multiple paths:

```yaml
on:
  push:
    paths:
      - 'frontend/**'
      - 'backend/**'
```

Meaning:

> Run if either frontend OR backend changes.

So:

```text
frontend/app.js → ✅
backend/app.py  → ✅
terraform/main.tf → ❌
README.md          → ❌
```

---

# 23. File extension patterns

You can also use patterns.

For example:

```yaml
on:
  push:
    paths:
      - '**.js'
```

This means files ending in `.js`.

For example:

```text
app.js       → ✅
script.js    → ✅
index.html   → ❌
style.css    → ❌
```

Another:

```yaml
paths:
  - '**.py'
```

means Python files.

---

# 24. A complete example

Here's a realistic workflow:

```yaml
name: Backend CI

on:
  push:
    branches:
      - main
      - develop

    paths:
      - 'backend/**'
      - '!backend/docs/**'

  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build
        run: echo "Building backend..."
```

Let's understand it:

### Push

```yaml
push:
  branches:
    - main
    - develop
```

Only:

```text
main
develop
```

### Paths

```yaml
paths:
  - 'backend/**'
  - '!backend/docs/**'
```

Backend changes trigger it, but the `backend/docs/` area is excluded.

### Pull Request

```yaml
pull_request:
  branches:
    - main
```

PRs targeting `main` trigger the workflow.

---

# 25. The easiest way to remember everything

Think about **WHAT happened** and **WHERE it happened**.

```text
                 GitHub Event
                      │
          ┌───────────┴───────────┐
          │                       │
       Branch                    Tag
          │                       │
    ┌─────┴─────┐           ┌─────┴─────┐
 branches  branches-ignore  tags  tags-ignore
          │
          │
        Files
          │
    ┌─────┴─────┐
  paths     paths-ignore
```

### Simple definitions

| Filter            | Meaning                                | Example      |
| ----------------- | -------------------------------------- | ------------ |
| `branches`        | Include specific branches              | `main`       |
| `branches-ignore` | Exclude branches                       | `develop`    |
| `tags`            | Include specific tags                  | `v*`         |
| `tags-ignore`     | Exclude tags                           | `beta-*`     |
| `paths`           | Include specific changed files/folders | `backend/**` |
| `paths-ignore`    | Exclude specific changed files/folders | `docs/**`    |

---

# 26. One practical example to remember for interviews

Suppose you have:

```text
project/
├── frontend/
├── backend/
├── docs/
└── terraform/
```

You want:

> "Run backend CI only when code is pushed to `main` and something inside `backend/` changes."

You write:

```yaml
name: Backend CI

on:
  push:
    branches:
      - main
    paths:
      - 'backend/**'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: echo "Backend CI running..."
```

The logic is:

```text
                    PUSH
                      │
                      ▼
                Is branch main?
                 /          \
               NO            YES
               ↓              ↓
              STOP      Did backend/** change?
                           /          \
                         NO            YES
                         ↓              ↓
                        STOP           RUN
```

That's the core idea of **event filters in GitHub Actions**.

### Interview answer

> **Event filters in GitHub Actions are conditions that control when a workflow should execute. For example, `branches` and `branches-ignore` filter based on branches, `tags` and `tags-ignore` filter based on Git tags, while `paths` and `paths-ignore` filter based on the files that were changed. They help us avoid unnecessary workflow executions and make CI/CD pipelines more efficient.**
