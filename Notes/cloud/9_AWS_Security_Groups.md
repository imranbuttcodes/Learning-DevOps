# 🔐 What is an AWS Security Group?

An **AWS Security Group (SG)** is basically a **virtual firewall for your EC2 instance**.

Its job is to control:

> **Which network traffic is allowed to reach your EC2 instance and which traffic is allowed to leave it.**

Think of your EC2 instance as a house:

```text
                 Internet
                    │
          ┌─────────▼─────────┐
          │   Security Group  │
          │  🛡️ Virtual       │
          │     Firewall      │
          └─────────┬─────────┘
                    │
                    ▼
              ┌───────────┐
              │    EC2    │
              │  Server   │
              └───────────┘
```

The Security Group stands **between the network and your EC2 instance**.

---

# 1. Why do we need it?

Imagine you launch an EC2 instance with a public IP:

```text
EC2 Public IP
     ↓
54.xxx.xxx.xxx
```

That IP is reachable from the Internet.

Without some kind of access control, you wouldn't want **every possible computer on the Internet** freely connecting to your server.

You might want:

```text
SSH      → only your laptop
HTTP     → everyone
HTTPS    → everyone
Database → nobody from Internet
```

Security Groups let you define these rules.

---

# 2. Inbound rules

**Inbound = traffic coming INTO your EC2.**

For example, suppose you want to SSH into your server.

SSH normally uses:

```text
TCP
Port 22
```

You could have:

```text
Inbound Rules

Protocol   Port    Source
──────────────────────────────
TCP        22      My IP
```

Meaning:

> "Allow SSH connections on port 22, but only from my IP address."

So:

```text
Your Laptop
     │
     │ TCP :22
     ▼
┌──────────────┐
│  Security    │
│    Group     │
└──────┬───────┘
       │
       ▼
      EC2
```

Your laptop gets through because its IP matches the rule.

Someone else:

```text
Random Computer
       │
       │ TCP :22
       ▼
 Security Group
       │
       ✖
    BLOCKED
```

---

# 3. Ports are important

A Security Group doesn't simply say:

> "Allow this computer."

It can control **specific network ports**.

For example:

|  Port | Common purpose                  |
| ----: | ------------------------------- |
|    22 | SSH                             |
|    80 | HTTP                            |
|   443 | HTTPS                           |
|  5050 | Your custom app                 |
|  8000 | Common FastAPI development port |
| 27017 | MongoDB                         |

Suppose later you deploy your FastAPI application on EC2:

```text
EC2
 │
 ├── :22    SSH
 ├── :80    HTTP
 └── :8000  FastAPI
```

You could create rules such as:

```text
22    → My IP only
80    → Anywhere
8000  → Maybe specific sources
```

That's much safer than opening everything.

---

# 4. What does `0.0.0.0/0` mean?

You'll see this **a lot** in AWS.

```text
0.0.0.0/0
```

means:

> **Any IPv4 address.**

So:

```text
Port 80
Source: 0.0.0.0/0
```

means:

> Anyone on the Internet can attempt to access port 80.

That's appropriate for a **public website**.

But:

```text
Port 22
Source: 0.0.0.0/0
```

means:

> Anyone on the Internet can attempt to connect to your SSH port.

That's generally something you want to restrict when possible.

---

# 5. Outbound rules

Now flip the direction.

**Outbound = traffic leaving your EC2.**

For example, your EC2 wants to install packages:

```text
EC2
 │
 │ HTTPS :443
 ▼
Internet
```

Or your application wants to call:

```text
EC2 → Groq API
EC2 → GitHub
EC2 → AWS APIs
EC2 → external services
```

Those are outbound connections.

A default Security Group commonly allows outbound traffic broadly, though you can restrict it if your architecture requires that.

---

# 6. The really important thing: Security Groups are stateful

This is a concept I want you to remember.

Suppose your EC2 makes a request:

```text
EC2 ───────────────→ Google
       HTTPS :443
```

Google sends the response:

```text
Google ─────────────→ EC2
          response
```

You don't normally need to create a separate inbound rule just for that response.

Why?

Because **Security Groups are stateful**.

AWS remembers that the connection was allowed.

So:

```text
Allowed outbound connection
          ↓
     Response traffic
          ↓
       Allowed
```

---

# 7. Security Group ≠ authentication

This distinction is VERY important.

Suppose your Security Group says:

```text
SSH :22
Source: 0.0.0.0/0
```

That does **NOT** mean everyone can log into your EC2.

It means:

> Everyone can **attempt to connect** to the SSH service.

They still need valid authentication.

For your EC2, you'll eventually use your SSH key:

```text
Internet
   │
   │ SSH connection
   ▼
Security Group
   │
   │ Port 22 allowed?
   ▼
EC2 SSH Server
   │
   │ Valid private key?
   ▼
Login
```

So there are multiple layers:

```text
Network access
      ↓
Security Group
      ↓
Application/service
      ↓
Authentication
      ↓
Authorization
```

---

# 8. Your current EC2 example

When you launched your EC2, AWS created something like:

```text
launch-wizard-1
```

with rules.

You specifically changed SSH from:

```text
SSH :22 → Anywhere
```

to:

```text
SSH :22 → My IP
```

So conceptually your server currently looks like:

```text
                    INTERNET
                       │
             ┌─────────┴─────────┐
             │   Security Group  │
             │                   │
             │  SSH :22          │
             │  Your IP only ✓   │
             │                   │
             │  HTTP :80          │
             │  ❌ Not allowed    │
             │                   │
             │  HTTPS :443        │
             │  ❌ Not allowed    │
             └─────────┬─────────┘
                       │
                       ▼
                 ┌───────────┐
                 │    EC2    │
                 │  Ubuntu   │
                 └───────────┘
```

That's a good setup for our **initial SSH lab**.

---

## 🧠 The mental model

Remember this one sentence:

> **A Security Group is a stateful virtual firewall that controls network traffic to and from an EC2 instance using rules based on things like protocol, port, and source/destination.**

And the simplest mental model:

```text
Security Group = EC2's network-level bouncer 🕵️

Inbound  → Who can come IN?
Outbound → Where can it GO?
```