Let's learn **Amazon Elastic Load Balancing (ELB)** properly from the concept first.

# ⚖️ Amazon Load Balancer

Imagine you have one FastAPI server:

```text
User
  │
  ▼
EC2
FastAPI
```

Everything works.

But then 1,000 users arrive:

```text
                ┌──→ EC2
Users ──────────┤
                └──→ 💥 overloaded
```

A single EC2 can become a bottleneck.

So we add a **Load Balancer**:

```text
                    ┌──→ EC2 #1
                    │
Users ──→ ALB ──────┼──→ EC2 #2
                    │
                    └──→ EC2 #3
```

The Load Balancer receives incoming requests and distributes them among multiple servers.

---

## What exactly is a Load Balancer?

A **load balancer is a managed service that receives network/application traffic and distributes that traffic across multiple backend targets.**

In AWS, this is provided through **Elastic Load Balancing (ELB)**.

Think:

> **ELB = AWS's load-balancing system**

It can distribute traffic across:

* EC2 instances
* containers
* IP addresses
* other supported targets

---

# Why do we need it?

### Without a load balancer

```text
                  ┌──→ EC2
                  │
Users ────────────┤
                  │
                  └──→ Single server
```

Problems:

* One server receives everything.
* If it goes down → application goes down.
* Scaling is harder.
* Traffic isn't distributed.

### With a load balancer

```text
                       ┌──→ EC2 #1
                       │
Users → Load Balancer ─┼──→ EC2 #2
                       │
                       └──→ EC2 #3
```

Now traffic can be distributed.

And if:

```text
EC2 #2 💀
```

the load balancer can stop sending traffic to it if its health checks show that it isn't healthy.

---

# 🏥 Health Checks

This is one of the most important concepts.

Suppose your FastAPI application exposes:

```text
GET /health
```

and returns:

```text
200 OK
```

The Load Balancer periodically checks your servers.

```text
ALB
 │
 ├──→ EC2 #1 → /health → 200 ✅
 │
 ├──→ EC2 #2 → /health → 200 ✅
 │
 └──→ EC2 #3 → /health → 500 ❌
```

The ALB can consider #3 unhealthy and stop routing normal traffic to it.

This is called a **health check**.

---

# What does "distribute" mean?

Suppose five requests arrive:

```text
Request 1
Request 2
Request 3
Request 4
Request 5
```

The load balancer might route them roughly like:

```text
             ALB
              │
       ┌──────┼──────┐
       ↓      ↓      ↓
      EC2-1  EC2-2  EC2-3

       ↓      ↓      ↓
      Req1   Req2   Req3
      Req4   Req5
```

The exact routing behavior depends on the load balancer type and configuration.

---

# 🔥 AWS has different load balancer types

You don't need to memorize everything.

The important ones are:

### 1. Application Load Balancer — ALB

Works at the **HTTP/HTTPS application layer**.

This is the one you should understand particularly well for your backend applications.

Example:

```text
api.example.com/users
api.example.com/products
```

ALB can make routing decisions based on things such as:

```text
/path
/host
HTTP headers
```

For example:

```text
             ALB
              │
       ┌──────┴──────┐
       │             │
 /api/users      /api/products
       │             │
       ↓             ↓
   User servers   Product servers
```

---

### 2. Network Load Balancer — NLB

Designed for **very high-performance TCP/UDP/TLS traffic** and lower-level network load balancing.

Conceptually:

```text
Client
  │
  ▼
 NLB
  │
  ├── EC2
  ├── EC2
  └── EC2
```

Don't worry about mastering NLB yet.

---

### 3. Gateway Load Balancer — GWLB

Used for deploying and scaling **network/security appliances**.

For example, specialized firewalls or inspection appliances.

Not a priority for your current learning path.

---

# ALB vs EC2

This distinction is important:

```text
ALB = traffic manager
EC2 = server that actually runs your application
```

The ALB does **not** replace your EC2.

Instead:

```text
Internet
    │
    ▼
   ALB
    │
    ├─────────┐
    ↓         ↓
  EC2 #1    EC2 #2
    │         │
  FastAPI   FastAPI
```

---

# And now connect this to what we've already learned

Remember our production architecture?

```text
User
 │
 ▼
Route 53
 │
 ▼
ALB
 │
 ├──→ EC2 #1
 ├──→ EC2 #2
 └──→ EC2 #3
       │
       ↓
     FastAPI
       │
       ├──→ RDS
       └──→ S3
```

Now the architecture makes much more sense:

* **Route 53** → DNS: "Where is my application?"
* **ALB** → distributes HTTP/HTTPS requests
* **EC2** → runs your FastAPI application
* **RDS** → database
* **S3** → object/file storage
* **IAM Role** → gives EC2 controlled AWS permissions
* **Auto Scaling Group** → creates/removes EC2 instances based on demand

And the really nice combination is:

```text
                 ALB
                  │
          ┌───────┼───────┐
          ↓       ↓       ↓
        EC2-1   EC2-2   EC2-3
          ↑       ↑       ↑
          └──── Auto Scaling ────┘
```

**ALB distributes the traffic.**

**Auto Scaling manages the number of EC2 instances.**

Those two services work together to make the application scalable and resilient.

That's the core of AWS Load Balancing.
