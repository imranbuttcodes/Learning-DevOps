# 2. IaaS vs PaaS vs SaaS

Now that we understand the evolution from **dedicated servers → VPS → cloud**, these three models become very intuitive.

The central question is:

> **How much of the technology stack do YOU manage, and how much does the provider manage for you?**

---

## 1. Start with the stack

Any application ultimately sits on layers of technology:

```text
┌──────────────────────────┐
│       Your Application   │  ← You write this
├──────────────────────────┤
│       Runtime            │
├──────────────────────────┤
│       Operating System   │
├──────────────────────────┤
│       Virtualization     │
├──────────────────────────┤
│       Servers            │
├──────────────────────────┤
│       Storage            │
├──────────────────────────┤
│       Networking         │
├──────────────────────────┤
│       Physical Hardware  │
└──────────────────────────┘
```

Now imagine AWS, Microsoft Azure, or another provider says:

> "You don't necessarily need to manage all of these layers yourself."

That's where **IaaS, PaaS, and SaaS** come in.

---

# 2. IaaS — Infrastructure as a Service

**IaaS = Infrastructure as a Service**

The provider gives you fundamental computing infrastructure.

For example:

**AWS EC2**

You essentially say:

> "Give me a virtual server."

AWS gives you the infrastructure.

```text
AWS manages:
────────────────────
Physical hardware
Networking infrastructure
Data center
Virtualization
etc.

YOU manage:
────────────────────
Operating system
Runtime
Dependencies
Application
Data
Configuration
```

Conceptually:

```text
       IaaS
        │
┌───────┴────────┐
│    Your App    │ ← YOU
├────────────────┤
│    Runtime     │ ← YOU
├────────────────┤
│      OS        │ ← YOU
├────────────────┤
│ Virtualization │ ← AWS
├────────────────┤
│    Hardware    │ ← AWS
└────────────────┘
```

### Examples

* AWS EC2
* Azure Virtual Machines
* Google Compute Engine

---

# 3. Why is it called Infrastructure?

Because you're basically renting **infrastructure**.

You get things like:

* CPU
* RAM
* Storage
* Networking
* Virtual machines

It's similar to renting a VPS, except cloud platforms give you much more flexibility and automation.

---

# 4. PaaS — Platform as a Service

Now imagine you tell the provider:

> "I don't want to manage the operating system either. Just give me an environment where I can deploy my application."

That's **PaaS**.

**PaaS = Platform as a Service**

The provider manages more of the stack.

```text
       PaaS
        │
┌───────┴────────┐
│    Your App    │ ← YOU
├────────────────┤
│    Runtime     │ ← Provider
├────────────────┤
│      OS        │ ← Provider
├────────────────┤
│ Infrastructure │ ← Provider
└────────────────┘
```

You mostly worry about:

> **"Here's my code. Run it."**

The provider handles much of the underlying infrastructure.

### Examples

Depending on the specific service/model:

* AWS Elastic Beanstalk
* Google App Engine
* Azure App Service
* Heroku

---

# 5. SaaS — Software as a Service

Now we go even further.

Instead of saying:

> "Give me a server."

or:

> "Give me a platform where I can deploy my application."

you say:

> **"Just give me the finished software."**

That's **SaaS**.

**SaaS = Software as a Service**

For example, when you use:

* Gmail
* Google Docs
* Slack
* Notion

you aren't managing:

```text
Server
OS
Runtime
Application infrastructure
Database
Networking
```

You simply use the software.

```text
       SaaS
        │
┌───────┴────────┐
│    Software    │ ← Provider manages it
├────────────────┤
│    Runtime     │
├────────────────┤
│      OS        │
├────────────────┤
│ Infrastructure │
└────────────────┘
```

Your responsibility is mostly:

> **Use the application and manage your account/data/configuration as applicable.**

---

# 6. The easiest way to remember it

Think about **pizza**. 🍕

### IaaS = ingredients

You get the basic infrastructure and make most of the pizza yourself.

### PaaS = partially prepared pizza

The provider handles more of the preparation; you focus on your application.

### SaaS = delivered pizza

You just consume the finished product.

---

# 7. Here's the really important comparison

```text
              YOU MANAGE                 PROVIDER MANAGES
                  │                              │
IaaS       ████████████████              ███████████
           App, Runtime, OS              Hardware, network,
                                        virtualization

PaaS       ████████                      ████████████████████
           Application                  Runtime, OS,
                                        infrastructure

SaaS       ██                            ███████████████████████
           Usage/config                 Almost everything
```

The more you move:

**IaaS → PaaS → SaaS**

the **less infrastructure you manage**.

And correspondingly, the provider manages more.

---

# 8. Let's connect this to what you've already learned

You built a Docker application.

Suppose you have:

```text
FastAPI
   ↓
Docker
   ↓
Linux
   ↓
Server
```

### Option A — IaaS

You rent an **EC2 instance**.

Then:

```text
EC2
 ↓
Ubuntu/Linux
 ↓
Docker
 ↓
Your container
 ↓
Your application
```

You manage a lot.

That's very close to the DevOps workflow you're learning.

---

### Option B — PaaS

You could use a platform where you provide your application/container and the platform handles much of:

* OS
* infrastructure
* deployment
* scaling

You focus primarily on your application.

---

### Option C — SaaS

Imagine you're not building the application at all.

You simply use something like **Notion**.

You don't care whether their backend is running on:

```text
EC2
Kubernetes
VMs
Bare metal
```

That's their problem.

You just use Notion.

---

# 9. What about AWS itself?

Here's something beginners often misunderstand:

> **AWS isn't just IaaS.**

AWS provides services across different levels.

For example:

| Service           | Rough category           |
| ----------------- | ------------------------ |
| EC2               | IaaS                     |
| S3                | Managed cloud service    |
| RDS               | Managed database service |
| Lambda            | Serverless / FaaS        |
| Elastic Beanstalk | PaaS-like                |
| CloudWatch        | Managed service          |

So **AWS is a cloud platform containing many different service models**, not simply "an IaaS provider."

---

# 10. Why would you choose one over another?

Imagine you're building an AI API.

### IaaS

You want maximum control:

```text
EC2
 ↓
Linux
 ↓
Docker
 ↓
FastAPI
 ↓
AI application
```

You can customize almost everything.

**More control → more responsibility.**

---

### PaaS

You mainly care about:

```text
Your application
       ↓
Deploy
       ↓
Platform handles infrastructure
```

**Less infrastructure work → less control.**

---

### SaaS

You're not building the infrastructure or application yourself.

You simply consume an existing product.

**Least management → least control/customization.**

---

# 11. The trade-off

This is the deeper concept.

```text
             CONTROL
                ↑
                │
              IaaS
                │
                │
              PaaS
                │
                │
              SaaS
                │
                ↓
        LESS MANAGEMENT
```

Generally:

**More control = more responsibility**

**Less control = less infrastructure responsibility**

Neither is universally "better." It depends on what you're trying to accomplish.

---

# 12. One more term: FaaS

You'll probably encounter this in your AWS course.

**FaaS = Function as a Service**

AWS Lambda is the classic example.

Instead of managing a server:

```text
Server
 ↓
OS
 ↓
Runtime
 ↓
Application
```

you essentially provide a function:

```text
Event
  ↓
Lambda function
  ↓
Result
```

AWS handles the underlying infrastructure and automatically manages execution resources.

This is commonly called **serverless computing**.

We'll go deeper into that when your AWS course reaches Lambda.

---

# 🧠 The mental model you should retain

Don't memorize random definitions.

Remember this:

```text
               WHO MANAGES WHAT?

IaaS
You:        Application + Runtime + OS
Provider:   Infrastructure

        ↓ more provider management

PaaS
You:        Application
Provider:   Runtime + OS + Infrastructure

        ↓ more provider management

SaaS
You:        Use the software
Provider:   Almost the entire technology stack
```

And connect it to your own path:

```text
Your AI application
       │
       ├── IaaS → EC2 + Docker
       │
       ├── PaaS → managed application platform
       │
       └── SaaS → consume someone else's AI/software service
```

