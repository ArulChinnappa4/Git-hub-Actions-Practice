

# What is an Activity Type in GitHub Actions?

An **event** tells GitHub Actions **what kind of GitHub object/action caused the workflow to run**.

For example:

```yaml
on:
  pull_request:
```

This says:

> Run the workflow when something happens to a Pull Request.

But a Pull Request can have many different activities:

* PR opened
* PR closed
* PR updated with new commits
* PR assigned to someone
* PR labeled
* PR edited
* PR reopened
* etc.

These individual activities are called **activity types**.

You can control them using:

```yaml
types:
```

---

# 1. Basic Structure

Without activity types:

```yaml
on:
  pull_request:
```

The workflow can run for the default pull-request activity types.

With activity types:

```yaml
on:
  pull_request:
    types:
      - opened
      - closed
```

Now you're saying:

> Run this workflow when a PR is **opened** or **closed**.

---

# 2. Think of it this way

Imagine a Pull Request:

```text
                 Pull Request
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
    opened          closed         synchronize
       │               │                │
       ↓               ↓                ↓
   Activity         Activity         Activity
    Type             Type             Type
```

So:

```text
Event = Pull Request

Activity type = What happened to that Pull Request?
```

---

# 3. `opened`

`opened` means:

> A new Pull Request was created.

Example:

You have:

```text
main
  ↑
  │
feature/login
```

You create:

```text
feature/login
       ↓
Create Pull Request
       ↓
main
```

GitHub generates:

```text
pull_request
activity type = opened
```

Workflow:

```yaml
name: PR Opened

on:
  pull_request:
    types:
      - opened

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "A new Pull Request was opened"
```

### What happens?

```text
Create PR
   ↓
opened
   ↓
Workflow runs ✅
```

---

# 4. `synchronize`

This one is **very important**.

`synchronize` happens when **new commits are pushed to the branch associated with an existing Pull Request**.

Example:

You create:

```text
feature/login
```

Then create PR:

```text
feature/login → main
```

The PR is now open.

Then you modify your code:

```bash
git add .
git commit -m "Fix login bug"
git push
```

Now the existing PR gets updated.

GitHub generates:

```text
pull_request
activity type = synchronize
```

Example workflow:

```yaml
name: PR Tests

on:
  pull_request:
    types:
      - opened
      - synchronize

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: echo "Running tests..."
```

Now:

```text
Create PR
    ↓
opened
    ↓
Workflow runs ✅

Push another commit
    ↓
synchronize
    ↓
Workflow runs ✅
```

This is commonly used for **CI testing**.

---

# 5. `closed`

`closed` means:

> The Pull Request was closed.

A PR can be closed in two ways:

### Case 1 — Merged

```text
PR
 ↓
Merge
 ↓
Closed
```

### Case 2 — Closed without merging

```text
PR
 ↓
Close
 ↓
Closed
```

Both produce:

```text
activity type = closed
```

Example:

```yaml
on:
  pull_request:
    types:
      - closed
```

Now the workflow runs when the PR closes.

---

# 6. Important: `closed` doesn't necessarily mean merged

This is a common interview question.

If:

```yaml
on:
  pull_request:
    types:
      - closed
```

it means:

> The PR was closed.

It does **not automatically mean it was merged**.

If you specifically want to know whether it was merged, you can use:

```yaml
if: github.event.pull_request.merged == true
```

Example:

```yaml
name: After Merge

on:
  pull_request:
    types:
      - closed

jobs:
  deploy:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest

    steps:
      - run: echo "PR was merged!"
```

The flow is:

```text
PR closed
   │
   ├── merged = true
   │       ↓
   │      RUN ✅
   │
   └── merged = false
           ↓
          SKIP ❌
```

---

# 7. `reopened`

Suppose a PR was closed:

```text
PR
 ↓
Closed
```

Later someone reopens it:

```text
Reopen PR
```

The activity type is:

```text
reopened
```

Example:

```yaml
on:
  pull_request:
    types:
      - reopened
```

Flow:

```text
Closed PR
    ↓
Reopen
    ↓
reopened
    ↓
Workflow runs
```

---

# 8. `assigned`

A Pull Request can be assigned to a person.

For example:

```text
Pull Request
     ↓
Assignee
     ↓
Arul
```

This creates:

```text
activity type = assigned
```

Example:

```yaml
on:
  pull_request:
    types:
      - assigned
```

Workflow runs when someone is assigned to the PR.

---

# 9. `unassigned`

The opposite of `assigned`.

If someone is removed as an assignee:

```text
PR
 ↓
Arul assigned
 ↓
Remove Arul
```

Activity:

```text
unassigned
```

Example:

```yaml
on:
  pull_request:
    types:
      - unassigned
```

---

# 10. `labeled`

A PR can have labels.

For example:

```text
Pull Request

Labels:
  bug
  urgent
  backend
```

When a label is added:

```text
activity type = labeled
```

Example:

```yaml
on:
  pull_request:
    types:
      - labeled
```

Now:

```text
Add "bug" label
       ↓
labeled
       ↓
Workflow runs
```

---

# 11. `unlabeled`

Opposite of `labeled`.

If:

```text
PR
 ↓
bug label
 ↓
Remove bug label
```

Activity:

```text
unlabeled
```

Example:

```yaml
on:
  pull_request:
    types:
      - unlabeled
```

---

# 12. `edited`

`edited` means information about the Pull Request was changed.

For example, you create:

```text
Title:
Fix login bug
```

Then change it to:

```text
Fix login authentication bug
```

That's an edit.

Other PR information can also be edited.

Example:

```yaml
on:
  pull_request:
    types:
      - edited
```

---

# 13. `converted_to_draft`

A Pull Request can be converted from a normal PR to a draft PR.

Example:

```text
Ready PR
   ↓
Convert to draft
   ↓
converted_to_draft
```

Workflow:

```yaml
on:
  pull_request:
    types:
      - converted_to_draft
```

---

# 14. `ready_for_review`

The opposite situation.

Suppose you have:

```text
Draft PR
```

After finishing your work:

```text
Ready for review
```

Activity:

```text
ready_for_review
```

Example:

```yaml
on:
  pull_request:
    types:
      - ready_for_review
```

---

# 15. `review_requested`

Suppose you create a PR and request a review from another developer.

```text
PR
 ↓
Request review
 ↓
Developer A
```

Activity:

```text
review_requested
```

Example:

```yaml
on:
  pull_request:
    types:
      - review_requested
```

---

# 16. `review_request_removed`

If the review request is removed:

```text
Request review
      ↓
Remove reviewer
      ↓
review_request_removed
```

Example:

```yaml
on:
  pull_request:
    types:
      - review_request_removed
```

---

# 17. `milestoned`

GitHub allows you to associate a Pull Request with a **milestone**.

For example:

```text
Milestone:
Release v1.0
```

When a PR is added to that milestone:

```text
milestoned
```

---

# 18. `demilestoned`

If the milestone is removed:

```text
Milestone removed
       ↓
demilestoned
```

---

# 19. `auto_merge_enabled`

GitHub supports automatic merging.

If auto-merge is enabled for a PR:

```text
auto_merge_enabled
```

You can trigger a workflow based on this activity.

---

# 20. A Practical Example

Suppose you're working on your **BMW-SPAREHUB** project.

Your workflow needs to run tests when:

1. PR is opened
2. New commits are pushed
3. PR is reopened

You can write:

```yaml
name: Pull Request CI

on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

Now:

```text
                  Pull Request
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     opened       synchronize      reopened
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                  Run CI Tests
```

But if someone:

```text
adds a label
```

then:

```text
labeled
   ↓
Workflow doesn't run ❌
```

because `labeled` isn't included.

---

# 21. Activity Type + Branch Filter

You can combine activity types with branch filters.

Example:

```yaml
name: Main PR Validation

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize
      - reopened

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: npm test
```

This means:

> Run when a PR targeting `main` is opened, updated with new commits, or reopened.

Think:

```text
                Pull Request
                     │
                     ↓
              Target branch?
                     │
                    main
                     │
                     ↓
             Activity type?
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       opened   synchronize  reopened
          │          │          │
          └──────────┼──────────┘
                     ↓
                  RUN ✅
```

---

# 22. Activity Type vs Event Filter

This distinction is **very important for interviews**.

### Event

```yaml
on:
  pull_request:
```

Means:

> Something happened to a Pull Request.

### Activity type

```yaml
types:
  - opened
  - closed
```

Means:

> Only run for these specific Pull Request activities.

### Branch filter

```yaml
branches:
  - main
```

Means:

> Only consider Pull Requests targeting `main`.

### Path filter

```yaml
paths:
  - 'backend/**'
```

Means:

> Only consider it when relevant files changed.

You can combine them:

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize
    paths:
      - 'backend/**'
```

Meaning:

> Run when a PR targeting `main` is opened or receives new commits, and the relevant changed files are under `backend/`.

---

# 23. Easy way to remember

Think about a Pull Request as a **person's application**.

The **event** is:

> "An application is being processed."

The **activity type** is:

> "What happened to the application?"

For a PR:

```text
pull_request
│
├── opened              → PR created
├── synchronize         → New commit pushed
├── closed              → PR closed
├── reopened            → PR reopened
├── assigned            → Person assigned
├── unassigned          → Person removed
├── labeled             → Label added
├── unlabeled           → Label removed
├── edited              → PR information changed
├── review_requested    → Reviewer requested
├── review_request_removed
├── ready_for_review    → Draft → Ready
├── converted_to_draft  → Ready → Draft
├── milestoned          → Milestone added
└── demilestoned        → Milestone removed
```

### Interview answer

> **Activity types in GitHub Actions allow us to specify the particular action within an event that should trigger a workflow. For example, the `pull_request` event can have activity types such as `opened`, `synchronize`, `closed`, `reopened`, `assigned`, and `labeled`. We use the `types` keyword to select specific activities. This gives us more precise control over when a workflow runs.**
