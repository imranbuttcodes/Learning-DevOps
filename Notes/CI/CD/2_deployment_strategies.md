# Deployment Strategies:

They answer one core question:

> **"How do I release a new version of my application without unnecessarily taking the service down or risking all users at once?"**

They fit **after CI/CD fundamentals**, because CI/CD gets the new version built; deployment strategy determines **how that version reaches users**.

---

# 1. Rolling Deployment

This is probably the easiest one to understand.

Suppose you have **4 application instances**:

```text
Current version:

[ v1 ] [ v1 ] [ v1 ] [ v1 ]
```

Instead of replacing everything simultaneously, you gradually replace them:

```text
Step 1:
[ v2 ] [ v1 ] [ v1 ] [ v1 ]

Step 2:
[ v2 ] [ v2 ] [ v1 ] [ v1 ]

Step 3:
[ v2 ] [ v2 ] [ v2 ] [ v1 ]

Step 4:
[ v2 ] [ v2 ] [ v2 ] [ v2 ]
```

Users can continue using the application while the update happens.

### Mental model

> **Replace instances gradually.**

### Advantage

No need to take the entire application offline.

### Trade-off

During the rollout, **v1 and v2 may coexist**.

That means your new version needs to be compatible with the old version during the transition.

---

# 2. Blue-Green Deployment

Here we maintain **two complete environments**.

```text
BLUE
[v1] [v1] [v1]
       │
       │ currently receiving traffic
       ▼
     USERS
```

We build the new version separately:

```text
BLUE                         GREEN

[v1] [v1] [v1]              [v2] [v2] [v2]
     │                            │
     └────── USERS ───────────────┘
             ↑
        traffic currently
        goes to BLUE
```

We test GREEN.

If everything looks good:

```text
Before:

Users
  ↓
BLUE (v1)

GREEN (v2)
```

Switch traffic:

```text
After:

Users
  ↓
GREEN (v2)

BLUE (v1)
```

If there's a problem, traffic can potentially be switched back to BLUE.

### Mental model

> **Keep two environments and switch traffic between them.**

### Advantage

Very clean rollback concept.

### Trade-off

You're maintaining two environments, so it can require more infrastructure/resources.

---

# 3. Canary Deployment

This one is different.

Instead of sending **everyone** to v2, we send a **small percentage** of traffic to it first.

For example:

```text
             Users
               │
        ┌──────┴──────┐
        │             │
       95%            5%
        │             │
        ▼             ▼
       v1             v2
```

We monitor v2.

If everything looks good:

```text
95% / 5%
   ↓
80% / 20%
   ↓
50% / 50%
   ↓
0% / 100%
```

If v2 has serious problems, we can stop the rollout before everybody is affected.

### Mental model

> **Expose the new version to a small portion of users first.**

---

# The key difference

This is what I want you to remember:

| Strategy       | Main idea                                                |
| -------------- | -------------------------------------------------------- |
| **Rolling**    | Replace instances gradually                              |
| **Blue-Green** | Run old + new environments, then switch traffic          |
| **Canary**     | Give the new version a small percentage of traffic first |

Think:

```text
ROLLING
v1 → v2 → v2 → v2
     gradual replacement


BLUE-GREEN
BLUE(v1) ← traffic switch → GREEN(v2)


CANARY
v1 ───────── 95% traffic
v2 ─────────  5% traffic
                 ↓
             gradually increase
```

## Where does this fit in our roadmap?

I'd put it here:

```text
CI/CD
  ↓
GitHub Actions
  ↓
Deployment strategies
  ├── Rolling
  ├── Blue-Green
  └── Canary
  ↓
AWS
  ↓
Actual deployment
```

And importantly: **Kubernetes will make these strategies much more relevant later**, especially rolling deployments.
