

```text
GitHub Repository
       │
       ├── Variables
       │      ↓
       │   ${{ vars.MY_VAR }}
       │
       └── Secrets
              ↓
           ${{ secrets.MY_SECRET }}
```

**Variables** are for normal configuration values.
**Secrets** are for sensitive values such as passwords, API keys, tokens, etc.

---

# Part 1 — Create a Repository Variable

Let's create:

```text
Name:  MY_VAR
Value: Hello from GitHub
```

### Step 1 — Open your repository

Go to your GitHub repository.

For example:

```text
github.com/your-username/your-repository
```

---

### Step 2 — Open Settings

At the top of your repository, click:

**Settings**

You should see something like:

```text
Code
Issues
Pull requests
Actions
Projects
Wiki
Security
Insights
Settings
```

Click **Settings**.

---

### Step 3 — Find Secrets and variables

On the left-hand side, find:

**Secrets and variables**

Expand it.

You should see:

```text
Secrets and variables
    ├── Actions
```

Click **Actions**.

---

### Step 4 — Open Variables

You'll see two tabs:

```text
Secrets
Variables
```

Select:

**Variables**

---

### Step 5 — Create a new variable

Click:

**New repository variable**

Enter:

```text
Name:
MY_VAR

Value:
Hello from GitHub
```

Then click:

**Add variable**

Your repository variable is now created.

---

# Part 2 — Use the Variable in GitHub Actions

Now create/edit your workflow:

```text
.github/
└── workflows/
    └── contexts.yml
```

Use:

```yaml
name: Contexts Demo

on:
  push:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Get variable
        run: |
          echo "Variable value: ${{ vars.MY_VAR }}"
```

Commit and push the workflow.

GitHub Actions will run it.

You should see:

```text
Variable value: Hello from GitHub
```

So the flow is:

```text
GitHub Settings
      ↓
Variables
      ↓
MY_VAR = Hello from GitHub
      ↓
Workflow
      ↓
${{ vars.MY_VAR }}
      ↓
Hello from GitHub
```

---

# Part 3 — Create a Secret

Now let's create a secret.

For learning, let's use:

```text
Name:
MY_SECRET

Value:
my-secret-value-123
```

**In a real project, don't use a real password/token for this demo.**

---

### Step 1 — Go back to

```text
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
```

---

### Step 2 — Select Secrets

You'll see:

```text
Secrets
Variables
```

Click:

**Secrets**

---

### Step 3 — Create repository secret

Click:

**New repository secret**

Enter:

```text
Name:
MY_SECRET

Secret:
my-secret-value-123
```

Then click:

**Add secret**

---

# Part 4 — Use the Secret in the Workflow

Now modify your workflow:

```yaml
name: Variables and Secrets

on:
  push:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Get variable
        run: |
          echo "Variable: ${{ vars.MY_VAR }}"

      - name: Use secret
        run: |
          echo "Secret is configured"
```

Notice that I **didn't print the actual secret**.

You can use the secret in commands without displaying it.

For example:

```yaml
- name: Use secret
  run: |
    some-command
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
```

Inside your command, the environment variable is available as:

```bash
$MY_SECRET
```

For example:

```yaml
- name: Use secret
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
  run: |
    echo "Secret is available to this step"
```

---

# Part 5 — Important Difference

You might wonder:

> Why don't we simply put everything in `env`?

Because `env` and GitHub `secrets` serve different purposes.

### Variable

```text
MY_VAR = Hello from GitHub
```

It's normal configuration.

Use:

```yaml
${{ vars.MY_VAR }}
```

### Secret

```text
MY_SECRET = sensitive-value
```

It's sensitive information.

Use:

```yaml
${{ secrets.MY_SECRET }}
```

---

# Part 6 — Real Example: Docker Hub

This is particularly relevant to your Docker/GitHub Actions learning.

Suppose you want GitHub Actions to push your Docker image to Docker Hub.

You shouldn't write:

```yaml
username: myusername
password: mypassword
```

❌ Don't do this.

Instead create:

```text
Variables:

DOCKER_USERNAME
```

and:

```text
Secrets:

DOCKER_PASSWORD
```

Then:

```yaml
name: Docker Build and Push

on:
  push:
    branches:
      - main

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Docker Login
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Image
        run: docker build -t my-calculator .

      - name: Push Image
        run: docker push my-calculator
```

Here:

```text
DOCKER_USERNAME
      ↓
vars.DOCKER_USERNAME

DOCKER_PASSWORD
      ↓
secrets.DOCKER_PASSWORD
```

---

# Part 7 — Another Important Method: Environment Variables

You can also pass a secret into a command through `env`.

For example:

```yaml
steps:
  - name: Login
    env:
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
    run: |
      echo "Docker password is available to this step"
```

Inside the shell:

```bash
$DOCKER_PASSWORD
```

contains the secret.

This is often cleaner than putting the expression directly into a command.

---

# Part 8 — Repository vs Environment vs Organization

GitHub gives you different places to store variables and secrets.

You'll commonly encounter:

```text
Repository
Organization
Environment
```

For your current learning, start with:

### Repository secrets

Available to workflows in that repository.

```text
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
Secrets
```

### Repository variables

Same location:

```text
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
Variables
```

Later, you can learn **Environment secrets/variables**, which are useful for things like:

```text
development
staging
production
```

For example:

```text
production
   ├── AWS_ACCESS_KEY
   ├── AWS_SECRET_KEY
   └── DATABASE_PASSWORD
```

---

# Part 9 — Complete Learning Workflow

I recommend creating this exact workflow to practice.

```yaml
name: Variables and Secrets Demo

run-name: Variables and Secrets Demo

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Display variable
        run: |
          echo "Application: ${{ vars.MY_VAR }}"

      - name: Use secret
        env:
          MY_SECRET: ${{ secrets.MY_SECRET }}
        run: |
          echo "Secret has been provided to this step"
          echo "Secret length: ${#MY_SECRET}"
```

Create these in GitHub:

### Variable

```text
MY_VAR
```

Value:

```text
Hello from GitHub
```

### Secret

```text
MY_SECRET
```

Value:

```text
my-secret-value-123
```

Then manually run the workflow.

You'll see:

```text
Application: Hello from GitHub
Secret has been provided to this step
Secret length: 19
```

The actual secret isn't displayed.

---

# Part 10 — Don't confuse these three

This is very important for your GitHub Actions learning:

### `env`

Defined in your YAML:

```yaml
env:
  APP_NAME: Calculator
```

Use:

```yaml
${{ env.APP_NAME }}
```

---

### `vars`

Created in GitHub **Variables**:

```text
MY_VAR = Hello
```

Use:

```yaml
${{ vars.MY_VAR }}
```

---

### `secrets`

Created in GitHub **Secrets**:

```text
MY_SECRET = sensitive-value
```

Use:

```yaml
${{ secrets.MY_SECRET }}
```

So remember:

```text
              GitHub Actions
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
      env          vars        secrets
       │            │            │
   YAML file     GitHub       GitHub
                 Variables     Secrets
       │            │            │
       ↓            ↓            ↓
  env.NAME      vars.NAME   secrets.NAME
```

### Interview answer

> **GitHub Actions variables are used to store reusable configuration values, while secrets are used to securely store sensitive information such as passwords, tokens, and API keys. Repository variables and secrets can be created under Settings → Secrets and variables → Actions. In a workflow, variables are accessed using `${{ vars.NAME }}` and secrets using `${{ secrets.NAME }}`. Secrets should not be hardcoded in the workflow file.**
