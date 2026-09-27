

# 1. What is a Workflow Context?

A **context** is a collection of information that GitHub makes available to your workflow while it is running.

Think of it like a set of **objects containing useful information** about the current workflow.

For example:

```yaml
${{ github.repository }}
```

can give you the repository name.

And:

```yaml
${{ github.actor }}
```

can give you the GitHub user who triggered the workflow.

So:

```text
GitHub Actions workflow
        │
        ├── github context
        ├── env context
        ├── vars context
        ├── secrets context
        ├── inputs context
        ├── matrix context
        └── needs context
```

Each context provides a different type of information.

---

# 2. Why do we need contexts?

Imagine your workflow is running in a repository:

```text
Arul/calculator
```

and someone pushes code to:

```text
main
```

Your workflow may need to know:

* Who triggered the workflow?
* Which repository is this?
* Which branch was pushed?
* Which commit triggered it?
* What environment variables are available?
* What secrets are configured?
* Which matrix configuration is currently running?
* Did a previous job succeed?
* What inputs did the user provide?

Instead of hardcoding these values, GitHub provides them through **contexts**.

For example, don't do:

```yaml
run: echo "Arul/calculator"
```

You can dynamically do:

```yaml
run: echo "${{ github.repository }}"
```

Now the same workflow can work in another repository.

---

# 3. Basic syntax

Most contexts are accessed using:

```yaml
${{ context.property }}
```

For example:

```yaml
${{ github.actor }}
```

Break it down:

```text
${{ github.actor }}
     │       │
     │       └── Property
     │
     └── Context
```

Another:

```yaml
${{ secrets.DOCKER_PASSWORD }}
```

```text
${{ secrets.DOCKER_PASSWORD }}
     │       │
     │       └── Secret name
     │
     └── Context
```

This `${{ }}` syntax is called a **GitHub Actions expression**.

---

# 4. `github` Context

The `github` context contains information about the **workflow run and GitHub event**.

This is probably the context you'll use most frequently.

Example:

```yaml
name: GitHub Context Demo

on:
  push:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Show information
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Actor: ${{ github.actor }}"
          echo "Branch: ${{ github.ref }}"
          echo "Commit: ${{ github.sha }}"
```

Suppose:

```text
Repository = Arul/calculator
Actor      = Arul
Branch     = refs/heads/main
Commit     = abc123...
```

Then GitHub substitutes those values when executing the workflow.

---

## Important `github` properties

### `github.actor`

Who triggered the workflow?

```yaml
${{ github.actor }}
```

Example:

```text
Arul
```

---

### `github.repository`

Repository name:

```yaml
${{ github.repository }}
```

Example:

```text
Arul/calculator
```

---

### `github.ref`

The Git reference that triggered the workflow.

For example:

```text
refs/heads/main
```

for a branch.

For a tag:

```text
refs/tags/v1.0.0
```

---

### `github.sha`

Commit SHA that triggered the workflow.

Example:

```text
a82f7d91...
```

Useful when you want to identify exactly which commit is being built.

---

### `github.event_name`

Tells you which event triggered the workflow.

```yaml
${{ github.event_name }}
```

Could be:

```text
push
```

or:

```text
pull_request
```

or:

```text
workflow_dispatch
```

---

# 5. `github.event`

This is especially useful.

The `github.event` context contains information about the **actual event payload** that triggered the workflow.

For example:

```yaml
on:
  pull_request:
    types:
      - opened
```

You can access information about the PR:

```yaml
run: echo "${{ github.event.pull_request.title }}"
```

If the PR title is:

```text
Fix login bug
```

then:

```text
github.event.pull_request.title
```

contains:

```text
Fix login bug
```

You can also access:

```yaml
${{ github.event.pull_request.number }}
```

to get the PR number.

Think:

```text
github
  │
  ├── repository
  ├── actor
  ├── ref
  ├── sha
  ├── event_name
  └── event
        └── pull_request
              ├── title
              ├── number
              └── ...
```

---

# 6. `env` Context

`env` contains **environment variables**.

You can define them at different levels.

### Workflow level

```yaml
name: Demo

on: push

env:
  APP_NAME: Calculator
  ENVIRONMENT: production

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: |
          echo "App: ${{ env.APP_NAME }}"
          echo "Environment: ${{ env.ENVIRONMENT }}"
```

Output:

```text
App: Calculator
Environment: production
```

---

## Job-level environment variable

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    env:
      APP_NAME: Calculator

    steps:
      - run: echo "${{ env.APP_NAME }}"
```

---

## Step-level environment variable

```yaml
steps:
  - name: Build
    env:
      VERSION: "1.0"
    run: echo "${{ env.VERSION }}"
```

The variable exists for that step.

---

# 7. Why use `env`?

Instead of repeatedly writing:

```yaml
run: echo "production"
```

you can define:

```yaml
env:
  ENVIRONMENT: production
```

and use:

```yaml
${{ env.ENVIRONMENT }}
```

This makes workflows easier to maintain.

---

# 8. `vars` Context

`vars` contains **configuration variables** defined in GitHub.

For example, you can create a repository variable:

```text
APP_NAME = Calculator
```

Then access it:

```yaml
${{ vars.APP_NAME }}
```

Example:

```yaml
name: Variables Demo

on: push

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Application: ${{ vars.APP_NAME }}"
```

---

# 9. `env` vs `vars`

This is important.

### `env`

Defined in the workflow:

```yaml
env:
  APP_NAME: Calculator
```

Access:

```yaml
${{ env.APP_NAME }}
```

### `vars`

Configured in GitHub repository/organization/environment settings:

```text
Repository Settings
      ↓
Variables
      ↓
APP_NAME = Calculator
```

Access:

```yaml
${{ vars.APP_NAME }}
```

Simple distinction:

```text
env
 ↓
Environment variables defined for workflow/job/step

vars
 ↓
Configuration variables stored in GitHub
```

---

# 10. `secrets` Context

`secrets` is used for **sensitive information**.

Examples:

```text
Docker Hub password
AWS credentials
API keys
Tokens
Database passwords
```

You should **not** put these directly in your YAML.

Bad:

```yaml
env:
  PASSWORD: mypassword123
```

Instead, create a GitHub Secret:

```text
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
New repository secret
```

For example:

```text
DOCKER_PASSWORD
```

Then:

```yaml
${{ secrets.DOCKER_PASSWORD }}
```

Example:

```yaml
steps:
  - name: Login
    run: echo "${{ secrets.DOCKER_PASSWORD }}"
```

However, **don't print secrets in real workflows**. GitHub attempts to mask secret values in logs, but you should still avoid exposing them.

---

# 11. Real Docker example

Suppose you're pushing your calculator Docker image to Docker Hub.

You configure:

```text
DOCKER_USERNAME
DOCKER_PASSWORD
```

as GitHub Actions secrets.

Then:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Docker Login
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
```

Notice:

```text
secrets.DOCKER_USERNAME
secrets.DOCKER_PASSWORD
```

The workflow gets the sensitive values without putting them directly in the YAML.

---

# 12. `inputs` Context

`inputs` is used when a workflow receives **input values**.

The most common example is a manually triggered workflow.

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment to deploy"
        required: true
        type: choice
        options:
          - development
          - production
```

When you manually run the workflow, GitHub asks:

```text
Environment to deploy:

○ development
○ production
```

Suppose you choose:

```text
production
```

You can access it with:

```yaml
${{ inputs.environment }}
```

Example:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
```

Output:

```text
Deploying to production
```

---

# 13. Why `inputs` is useful

It allows you to make workflows **interactive and reusable**.

Instead of creating:

```text
deploy-dev.yml
deploy-prod.yml
```

you can create one:

```text
deploy.yml
```

and let the user choose:

```text
development
production
```

when starting the workflow.

---

# 14. `matrix` Context

This is one of the most useful concepts for CI.

Suppose your application needs to be tested against:

```text
Node.js 18
Node.js 20
Node.js 22
```

You could manually create three jobs.

But that's repetitive.

Instead:

```yaml
strategy:
  matrix:
    node-version:
      - 18
      - 20
      - 22
```

Then:

```yaml
${{ matrix.node-version }}
```

Example:

```yaml
name: Node Test

on: push

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version:
          - 18
          - 20
          - 22

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm ci

      - run: npm test
```

GitHub creates multiple job runs:

```text
                test
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Node 18     Node 20    Node 22
       │          │          │
      test       test       test
```

So:

```yaml
${{ matrix.node-version }}
```

means:

> Give me the current matrix value for this particular job.

---

# 15. `needs` Context

`needs` is used when one job **depends on another job**.

Example:

```text
Build
  ↓
Test
  ↓
Deploy
```

You can write:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

The important part:

```yaml
needs: build
```

means:

> Don't start `test` until `build` finishes successfully.

---

# 16. `needs` Context itself

The `needs` context allows you to access information about a job that this job depends on.

For example:

```yaml
${{ needs.build.result }}
```

This tells you the result of the `build` job.

Possible result:

```text
success
failure
cancelled
skipped
```

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build completed"

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - run: echo "Build result = ${{ needs.build.result }}"
```

Output:

```text
Build result = success
```

---

# 17. Why `needs` is useful in CI/CD

Consider a production pipeline:

```text
        Build
          ↓
        Test
          ↓
        Docker Build
          ↓
        Deploy
```

You don't want:

```text
Build ❌
  ↓
Deploy ❌
```

You want deployment only after previous jobs succeed.

So:

```yaml
deploy:
  needs:
    - build
    - test
    - docker
```

This creates:

```text
Build ─────┐
           │
Test ──────┼──→ Deploy
           │
Docker ────┘
```

---

# 18. `job` Context

There is also a `job` context.

It contains information about the current job.

For example:

```yaml
${{ job.status }}
```

can tell you the current job's status.

Possible values include:

```text
success
failure
cancelled
```

Example:

```yaml
steps:
  - run: echo "Job status is ${{ job.status }}"
```

---

# 19. `runner` Context

`runner` contains information about the machine/environment running your job.

Example:

```yaml
run: |
  echo "OS: ${{ runner.os }}"
  echo "Architecture: ${{ runner.arch }}"
```

You might get:

```text
OS: Linux
Architecture: X64
```

Other useful properties include:

```yaml
${{ runner.os }}
${{ runner.arch }}
${{ runner.name }}
```

This is useful when your workflow behaves differently depending on the operating system.

---

# 20. `strategy` Context

The `strategy` context contains information about the current job's strategy.

It's commonly used with matrix jobs.

For example, you might check:

```yaml
${{ strategy.job-index }}
```

to identify the current matrix job.

But as a beginner, focus primarily on:

```text
matrix
```

rather than `strategy`.

---

# 21. `steps` Context

The `steps` context lets you access information about **previous steps** in the same job.

You can give a step an ID:

```yaml
steps:

  - name: Generate value
    id: generate
    run: echo "version=1.0" >> "$GITHUB_OUTPUT"
```

Then another step can access its output:

```yaml
- name: Show value
  run: echo "${{ steps.generate.outputs.version }}"
```

The flow is:

```text
Step 1
  │
  │ id = generate
  │
  │ output = version
  ↓
Step 2
  │
  ↓
steps.generate.outputs.version
```

This is extremely useful for passing information between steps.

---

# 22. `needs` vs `steps`

This is an important distinction.

### `steps`

Used to access information from **previous steps in the same job**.

```yaml
${{ steps.build.outputs.version }}
```

### `needs`

Used to access information from **jobs that this job depends on**.

```yaml
${{ needs.build.outputs.version }}
```

Think:

```text
JOB
│
├── Step 1
│
├── Step 2
│
└── Step 3

steps → information between steps
```

Whereas:

```text
Job A
  ↓
Job B
  ↓
Job C

needs → information/dependency between jobs
```

---

# 23. `needs` with outputs

This is a very powerful real-world example.

Suppose Build generates a Docker image tag.

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.version.outputs.tag }}

    steps:
      - id: version
        run: echo "tag=v1.0.0" >> "$GITHUB_OUTPUT"
```

Then Deploy depends on Build:

```yaml
  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying ${{ needs.build.outputs.image-tag }}"
```

The information flows:

```text
Build Job
   │
   └── image-tag = v1.0.0
             │
             ↓
       needs.build.outputs
             │
             ↓
       Deploy Job
```

---

# 24. `github` vs `env` vs `vars` vs `secrets` vs `inputs`

This is probably the most important comparison.

| Context   | Purpose                         | Example                              |
| --------- | ------------------------------- | ------------------------------------ |
| `github`  | GitHub event/run information    | `${{ github.actor }}`                |
| `env`     | Environment variables           | `${{ env.NODE_ENV }}`                |
| `vars`    | GitHub configuration variables  | `${{ vars.APP_NAME }}`               |
| `secrets` | Sensitive values                | `${{ secrets.API_KEY }}`             |
| `inputs`  | Values supplied to workflow     | `${{ inputs.environment }}`          |
| `matrix`  | Current matrix combination      | `${{ matrix.node }}`                 |
| `needs`   | Information from dependent jobs | `${{ needs.build.result }}`          |
| `steps`   | Information from previous steps | `${{ steps.build.outputs.version }}` |
| `job`     | Current job information         | `${{ job.status }}`                  |
| `runner`  | Runner information              | `${{ runner.os }}`                   |

---

# 25. One Complete Corporate Example

Let's combine several contexts.

Imagine:

```text
BMW-SPAREHUB
```

You want a CI/CD workflow.

```yaml
name: BMW SpareHub CI/CD

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Deployment environment"
        required: true
        type: choice
        options:
          - staging
          - production

env:
  APP_NAME: bmw-sparehub

jobs:

  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version:
          - 20
          - 22

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Show information
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "User: ${{ github.actor }}"
          echo "Application: ${{ env.APP_NAME }}"
          echo "Node: ${{ matrix.node-version }}"

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install
        run: npm ci

      - name: Test
        run: npm test
```

Here:

### `github`

```yaml
${{ github.repository }}
${{ github.actor }}
```

Gives GitHub information.

### `env`

```yaml
${{ env.APP_NAME }}
```

Gives environment variable.

### `inputs`

```yaml
${{ inputs.environment }}
```

Gives the environment selected when manually running the workflow.

### `matrix`

```yaml
${{ matrix.node-version }}
```

Gives:

```text
20
```

or:

```text
22
```

depending on the current job.

---

# 26. Visualize all the contexts

This is the easiest mental model:

```text
                 GitHub Actions
                       │
                       ▼
                 Workflow Run
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
    github            env             vars
       │               │                │
 GitHub info       Environment      Configuration
       │
       ▼
    event
       │
       ▼
  push / PR / etc.


       ┌───────────────────────────────┐
       │                               │
       ▼                               ▼
    secrets                         inputs
       │                               │
 Sensitive values               User-provided values


       ┌───────────────────────────────┐
       │                               │
       ▼                               ▼
    matrix                           needs
       │                               │
 Multiple configurations        Previous jobs


       ┌───────────────────────────────┐
       │                               │
       ▼                               ▼
    steps                            runner
       │                               │
 Previous steps                 Machine information
```

---

# 27. The most important ones to learn first

Don't try to memorize every context immediately.

For your current GitHub Actions learning, I'd learn them in this order:

### Level 1 — Must know

```text
github
env
secrets
```

### Level 2 — Very important

```text
inputs
vars
matrix
needs
```

### Level 3 — Next

```text
steps
job
runner
strategy
```

---

# 28. One-line memory trick

Remember what each context **answers**:

```text
github
→ "What is happening in GitHub?"

env
→ "What environment variables do I have?"

vars
→ "What configuration variables are defined?"

secrets
→ "What sensitive values can I securely use?"

inputs
→ "What did the user provide?"

matrix
→ "Which configuration combination am I running?"

needs
→ "What happened to the job I depend on?"

steps
→ "What happened in my previous steps?"

job
→ "What is the status/info of my current job?"

runner
→ "What machine am I running on?"
```

### Interview answer

> **Workflow contexts in GitHub Actions are predefined collections of information that allow a workflow to access dynamic data during execution. For example, the `github` context provides information about the repository, event, branch, commit, and user; `secrets` provides securely stored sensitive values; `inputs` provides user-supplied workflow inputs; `matrix` provides the current matrix configuration; and `needs` provides information and outputs from dependent jobs. Contexts make workflows dynamic, reusable, and configurable instead of hardcoding values.**
