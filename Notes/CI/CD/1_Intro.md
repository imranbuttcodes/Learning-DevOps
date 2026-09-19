# CI/CD — Lesson 1: What problem are we solving?

Before touching GitHub Actions or YAML, we need the **mental model**.

You already know this workflow:

```text
You write code
     ↓
git add
     ↓
git commit
     ↓
git push
     ↓
GitHub
```

Now imagine you're working on a real backend.

Every time you push code, someone needs to:

```text
1. Get the latest code
2. Install dependencies
3. Run tests
4. Check whether the application builds
5. Build the Docker image
6. Push the image somewhere
7. Deploy it
```

Doing all of that **manually every time** becomes annoying and error-prone.

That's where **CI/CD** comes in.

---

# 1. What does CI mean?

**CI = Continuous Integration**

The key word is **integration**.

Imagine a team:

```text
Developer A ──┐
Developer B ──┼──→ GitHub repository
Developer C ──┘
```

Everyone is constantly pushing changes.

The problem is that changes can break the project.

For example:

```text
Developer A
    ↓
Changes FastAPI code
    ↓
git push
    ↓
Tests automatically run
    ↓
❌ Test failed
```

The team discovers the problem **immediately**, rather than discovering it days later.

So:

> **Continuous Integration means frequently integrating code changes into a shared repository while automatically validating those changes.**

Typically the validation includes:

```text
git push
   ↓
automated tests
   ↓
linting / checks
   ↓
build
   ↓
✅ or ❌
```

---

# 2. What does CD mean?

CD can mean two closely related things:

### Continuous Delivery

The software is automatically prepared and kept **ready for deployment**, but production deployment may require approval.

```text
Code
 ↓
Test
 ↓
Build
 ↓
Ready to deploy
 ↓
[Human approval]
 ↓
Production
```

### Continuous Deployment

The deployment itself is automated.

```text
Code
 ↓
Test
 ↓
Build
 ↓
Deploy
 ↓
Production
```

So the distinction is:

```text
Continuous Delivery
        ↓
"Ready to deploy"

Continuous Deployment
        ↓
"Actually deploy automatically"
```

---

# 3. CI/CD together

Put everything together:

```text
                    CI
                    │
                    ▼
git push → Test → Build
                    │
                    ▼
                   CD
                    │
                    ▼
              Deploy application
```

A more realistic version for **your FastAPI project**:

```text
You
 │
 │ git push
 ▼
GitHub
 │
 ▼
GitHub Actions
 │
 ├── Install Python
 ├── Install dependencies
 ├── Run tests
 ├── Build Docker image
 └── Push image to Docker Hub
                    │
                    ▼
                 Docker Hub
                    │
                    ▼
                 AWS EC2
                    │
                    ▼
              FastAPI container
```

That's ultimately what we're building toward.

---

# 4. Why is it called "Continuous"?

It doesn't mean:

> "The computer is continuously deploying every second."

😂

It means the process happens **frequently and automatically as development continues**.

For example:

```text
Monday
Developer pushes
     ↓
CI runs

Tuesday
Developer pushes
     ↓
CI runs

Wednesday
Developer pushes
     ↓
CI runs
```

Instead of waiting until the end of the project to discover that everything is broken.

---

# 5. Manual deployment vs CI/CD

### Without CI/CD

```text
Developer
   ↓
git push
   ↓
SSH into server
   ↓
git pull
   ↓
Install dependencies
   ↓
Run tests
   ↓
Build Docker image
   ↓
Restart application
   ↓
Check logs
```

And you repeat this every time.

### With CI/CD

```text
Developer
   ↓
git push
   ↓
GitHub Actions
   ↓
Tests
   ↓
Build
   ↓
Deploy
```

The human mainly does:

```bash
git push
```

That's the power of automation.

---

# 6. Now where does GitHub Actions fit?

This is important:

**CI/CD is not GitHub Actions.**

Think:

```text
CI/CD
  │
  │ is the practice/process
  ▼
GitHub Actions
  │
  │ is a tool/platform
  ▼
automates the process
```

Other platforms exist too:

* GitHub Actions
* GitLab CI/CD
* Jenkins
* CircleCI
* Azure Pipelines
* etc.

We're learning **GitHub Actions** because your code is already on GitHub and it integrates naturally with your Docker workflow.

---

# 7. The GitHub Actions mental model

This is the next thing you need to understand:

```text
GitHub Repository
       │
       │ event
       ▼
   Workflow
       │
       ▼
      Job
       │
       ▼
     Runner
       │
       ├── Step
       ├── Step
       ├── Step
       └── Step
```

We'll break these down next.

For now, remember:

> **A GitHub Actions workflow is an automated sequence of jobs and steps that runs when something happens in your repository.**

For example:

```text
git push
   ↓
workflow triggered
   ↓
runner starts
   ↓
checkout code
   ↓
install Python
   ↓
run tests
   ↓
build Docker image
```

---

## Your first mental model

Memorize this—not the YAML yet:

```text
┌──────────────────────────────┐
│          CI/CD               │
│                              │
│  CI                          │
│  ├─ Integrate code           │
│  ├─ Test                     │
│  └─ Build                    │
│                              │
│  CD                          │
│  ├─ Prepare release          │
│  └─ Deploy                   │
└──────────────────────────────┘
              │
              ▼
      GitHub Actions
              │
              ▼
       Automates it
```

### Next: **GitHub Actions architecture**

We'll go one level deeper into **Workflow → Event → Job → Runner → Step → Action**, because once those six things are clear, the YAML will stop looking like random syntax.
