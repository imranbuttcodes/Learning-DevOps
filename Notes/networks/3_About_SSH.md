# Lesson 3 — SSH: How You Actually Control a Server

Imagine you've rented an AWS EC2/VPS server.

It has:

```text
Public IP:
203.0.113.50
```

You are sitting on your laptop.

You want to run:

```text
docker ps
```

on that remote server.

How?

**SSH.**

---

## 1. What is SSH?

**SSH = Secure Shell.**

It lets you securely communicate with and control a remote computer over a network.

Conceptually:

```text
YOUR LAPTOP                         REMOTE SERVER

┌──────────────┐                    ┌──────────────┐
│              │                    │              │
│   Terminal   │ ───── SSH ──────► │    Linux     │
│              │                    │    Server    │
└──────────────┘                    └──────────────┘
```

After connecting, your terminal is effectively interacting with a shell running on the remote machine.

---

# 2. Where networking concepts appear

Suppose:

```text
Server IP = 203.0.113.50
SSH port  = 22
```

When you connect:

```text
ssh user@203.0.113.50
```

you're essentially saying:

> Connect me to the SSH service running at this server's network endpoint.

Break it down:

```text
203.0.113.50
       │
       └── IP → Which server?

22
       │
       └── Port → Which service?

SSH
       │
       └── Application protocol → How do we communicate?
```

And underneath SSH:

```text
SSH
 ↓
TCP
 ↓
IP
 ↓
Network
```

Now the networking lesson we just did has an actual purpose.

---

# 3. Why port 22?

SSH servers normally listen on:

```text
TCP port 22
```

So conceptually:

```text
203.0.113.50:22
        │
        ▼
   SSH service
```

But again:

**22 isn't magically "the SSH port."**

An administrator can configure SSH to listen somewhere else, such as:

```text
2222
```

So think:

> **22 is the conventional/default SSH port.**

---

# 4. What happens when you run SSH?

You type:

```text
ssh user@203.0.113.50
```

Conceptually:

```text
Your laptop
     │
     │ 1. Find/reach IP
     ▼
203.0.113.50
     │
     │ 2. TCP connection to port 22
     ▼
SSH server
     │
     │ 3. SSH protocol
     ▼
Authentication
     │
     ▼
Remote shell
```

And now:

```text
you@laptop:~$
```

becomes something like:

```text
user@server:~$
```

The prompt is your first clue that:

> **I'm no longer executing commands on my laptop.**

I'm executing them on the remote server.

---

# 5. How does SSH authenticate you?

There are two major approaches you'll encounter:

### Password authentication

```text
username
   +
password
```

### SSH key authentication

Much more important for DevOps.

You have:

```text
PRIVATE KEY
     │
     │ stays with you
     ▼
  Laptop
```

and the server has:

```text
PUBLIC KEY
     │
     ▼
~/.ssh/authorized_keys
```

Conceptually:

```text
YOUR MACHINE                    SERVER

Private key 🔑                  Public key 🔓
    │                               │
    └──────── SSH authentication ───┘
```

**The private key should remain private.**

You don't upload your private key to GitHub.

You don't send it to someone.

You don't paste it into Discord.

You don't put it inside your Docker image.

---

# 6. Why SSH keys are useful in DevOps

Imagine you're deploying an application.

You might have:

```text
GitHub
   ↓
CI/CD
   ↓
Cloud server
   ↓
Docker
   ↓
FastAPI
```

Automation may need to authenticate to the server.

SSH keys are one mechanism used for secure machine-to-machine access.

You'll encounter them constantly.

---

# 7. SSH and Linux

This is where our previous Linux learning comes back.

Once connected:

```text
ssh user@server
```

you can run:

```text
pwd
ls
cd
ps
systemctl
journalctl
docker
git
```

But now those commands are running **on the server**.

For example:

```text
Laptop
  │
  │ SSH
  ▼
Cloud VM
  │
  ├── Docker
  ├── Nginx
  ├── FastAPI
  └── PostgreSQL
```

That's basically how you'll manage Linux servers.

---

# 8. Now imagine deployment

Eventually we're going to do something like:

```text
                 INTERNET
                    │
                    ▼
              ┌───────────┐
              │ Cloud VM  │
              │           │
              │ Public IP │
              └─────┬─────┘
                    │
             ┌──────┴──────┐
             │             │
           :22           :443
             │             │
             ▼             ▼
            SSH           Nginx
                           │
                           ▼
                        :8000
                           │
                           ▼
                        FastAPI
```

Notice something important:

### SSH and HTTPS are two different services.

```text
:22  → SSH
:443 → HTTPS
```

That's why a server can simultaneously accept:

```text
SSH connection
+
web traffic
```

on different ports.

---

# 9. This also explains firewalls

Suppose your server firewall allows:

```text
22   ✅
80   ✅
443  ✅
8000 ❌
```

Then:

```text
SSH → works
HTTP → works
HTTPS → works
FastAPI :8000 → blocked externally
```

And that's actually what we **want** in many deployments.

We don't necessarily want users directly accessing:

```text
server:8000
```

Instead:

```text
Internet
   │
   ▼
HTTPS :443
   │
   ▼
Nginx
   │
   ▼
FastAPI :8000
```

FastAPI can remain inaccessible directly from the public internet.

**That's a real DevOps architecture decision.**

---

# 10. One very important distinction

Don't confuse:

```text
IP address
```

with:

```text
Port
```

or:

```text
Protocol
```

Think:

```text
             NETWORK ENDPOINT
                    │
             203.0.113.50:22
              /      |      \
             /       |       \
           IP       Port    Protocol
          where     which      how
                    service
```

For our example:

```text
203.0.113.50 → server
22           → SSH service
TCP          → transport
SSH          → communication protocol
```

That mental model will save you a **lot** of confusion later.

---

# Your DevOps mental model is forming

We've now connected:

```text
OSI
 │
 ├── L7 → HTTP / SSH
 │
 ├── L4 → TCP + ports
 │
 └── L3 → IP
```

And we're going to build upward from here:

```text
Linux
  ↓
SSH
  ↓
Docker
  ↓
Docker networking
  ↓
Nginx
  ↓
Cloud VM
  ↓
Firewall / Security Groups
  ↓
HTTPS
  ↓
CI/CD
  ↓
REAL DEPLOYMENT
```