# 1. What is Cloud Computing?

At its simplest:

> **Cloud computing means using computing resources over a network—usually the Internet—instead of owning and operating all the physical infrastructure yourself.**

Those resources can include:

* Compute (CPU/RAM)
* Storage
* Databases
* Networking
* Security
* Load balancing
* AI/ML services
* Monitoring
* etc.

For example, instead of buying a physical server and putting your FastAPI application on it:

```text
Your company
     │
     ▼
Own physical server
     │
     ├── CPU
     ├── RAM
     ├── SSD
     └── Network
          │
          ▼
       Your API
```

you can rent computing resources from a cloud provider:

```text
Your laptop
     │
     │ Internet
     ▼
   AWS
     │
     ▼
   EC2
     │
     ▼
 Your API
```

You don't physically own that EC2 server.

You're essentially saying:

> **"AWS, give me computing resources; I'll pay according to what I use."**

---

# 2. But why did we need the cloud?

This becomes much clearer if we look at the **evolution of hosting**.

Imagine you want to launch:

```text
example.com
     │
     ▼
Your web application
```

Your application needs a machine somewhere to run.

Historically, there were several ways to do this.

---

# 3. Dedicated Server — the old-school approach

Suppose you need a server.

You could literally rent an **entire physical machine** from a hosting company.

```text
              PHYSICAL SERVER
┌─────────────────────────────────┐
│                                 │
│       CPU: 16 cores             │
│       RAM: 64 GB                │
│       SSD: 1 TB                 │
│                                 │
│       Your application          │
│                                 │
└─────────────────────────────────┘
```

The entire machine belongs to you.

### Advantages

You get:

* Full control
* Dedicated CPU
* Dedicated RAM
* Dedicated storage
* Strong isolation
* Predictable performance

### Problem

You're probably not using all of it.

Suppose your application only needs:

```text
CPU: 2 cores
RAM: 4 GB
```

but you've rented:

```text
CPU: 16 cores
RAM: 64 GB
```

You're paying for a lot of unused capacity.

And if you need more capacity?

You need to get another physical server.

That's expensive and slow.

---

# 4. Shared Hosting

Hosting companies realized:

> "Why give one customer an entire server?"

Suppose a server has:

```text
CPU: 16 cores
RAM: 64 GB
```

They can put **many customers** on it.

```text
             ONE SERVER
┌─────────────────────────────────┐
│                                 │
│ Customer A                      │
│ Customer B                      │
│ Customer C                      │
│ Customer D                      │
│ Customer E                      │
│ Customer F                      │
│ ...                             │
│                                 │
└─────────────────────────────────┘
```

This is **shared hosting**.

It's cheap because you're sharing the underlying resources.

### Good for

* Small websites
* Blogs
* WordPress
* Personal websites
* Small businesses

### Problem

You don't have much control.

And resource usage from other customers can affect the environment.

You also typically can't configure the machine however you want.

---

# 5. VPS — Virtual Private Server

Now we get to a very important idea:

## Virtualization

Instead of having one physical server running one OS, we can use a **hypervisor** to divide the physical machine into virtual machines.

Imagine:

```text
              PHYSICAL SERVER
┌──────────────────────────────────┐
│                                  │
│          Hypervisor              │
│                                  │
├──────────┬──────────┬────────────┤
│   VPS A  │  VPS B   │   VPS C    │
│          │          │            │
│ 2 CPU    │ 4 CPU    │ 2 CPU      │
│ 4 GB RAM │ 8 GB RAM │ 4 GB RAM   │
└──────────┴──────────┴────────────┘
```

Each customer gets a **virtual machine**.

From the customer's perspective, it looks much more like they have their own server.

That's a **VPS (Virtual Private Server)**.

---

# 6. Why VPS was a big improvement

Compare:

### Shared hosting

```text
You
 ↓
Shared environment
 ↓
Limited control
```

### VPS

```text
You
 ↓
Virtual machine
 ↓
Your OS
 ↓
Your applications
```

Now you can typically:

* Install your own software
* Configure the OS
* Run your own backend
* Configure networking
* Install Docker
* Run databases
* Manage processes

Much more control.

And it's still cheaper than renting an entire physical server.

---

# 7. But VPS still has limitations

Imagine you rent:

```text
VPS:
4 CPU
8 GB RAM
100 GB SSD
```

Your application suddenly becomes extremely popular.

You now need:

```text
16 CPU
32 GB RAM
```

What happens?

You generally need to **upgrade the VPS** or create additional servers.

This can involve:

* Manual configuration
* Downtime depending on the provider/setup
* Capacity planning
* Managing infrastructure yourself

And you're still fundamentally renting a fixed slice of infrastructure.

---

# 8. Then came cloud computing

Cloud providers took virtualization + massive data centers + automation and built something much bigger.

Instead of simply saying:

> "Here's your VPS."

they said:

> **"Here is an API/control panel where you can provision computing resources whenever you need them."**

That's a major conceptual shift.

For example:

```text
You
 │
 │ API / Console
 ▼
AWS
 │
 ├── EC2
 ├── S3
 ├── RDS
 ├── VPC
 ├── Lambda
 ├── CloudFront
 └── dozens/hundreds of other services
```

You don't need to physically buy the hardware.

---

# 9. Cloud vs VPS

This distinction is important.

A VPS might give you:

> **One virtual server.**

Cloud gives you an ecosystem of **on-demand infrastructure and managed services**.

For example:

```text
Traditional VPS

        VPS
         │
    ┌────┴────┐
    │         │
   App       DB
```

Cloud architecture can become:

```text
                  Internet
                     │
                  Route 53
                     │
                Load Balancer
                     │
             ┌───────┼───────┐
             │       │       │
            EC2     EC2     EC2
             │       │       │
             └───────┼───────┘
                     │
                    RDS
                     │
                    S3
```

And these resources can be provisioned, configured, scaled and monitored programmatically.

---

# 10. The real revolution: Elasticity

This is one of the most important cloud concepts.

Suppose your application normally receives:

```text
100 requests/minute
```

But suddenly a marketing campaign causes:

```text
100,000 requests/minute
```

With traditional infrastructure, you had to **predict the required hardware ahead of time**.

Cloud infrastructure allows you to provision additional resources much more easily.

Conceptually:

```text
Normal traffic

       EC2
        │
       App


Huge traffic

       Load Balancer
        │
   ┌────┼────┬────┐
   │    │    │    │
  EC2   EC2  EC2  EC2
```

Then when traffic falls, you can reduce the resources again.

That's related to **elasticity**.

---

# 11. Scalability vs Elasticity

Don't mix these up.

### Scalability

The ability of a system to handle increasing workload by adding resources.

### Elasticity

The ability to **dynamically adjust resources according to demand**.

Example:

```text
Traffic
  │
  │          /\
  │         /  \
  │        /    \
  │_______/      \_______
           Time →
```

An elastic system can scale up during the spike and scale back down afterward.

---

# 12. Cloud isn't just "someone else's server"

This is a common beginner misconception.

People sometimes say:

> "Cloud is just someone else's computer."

There's some truth to the joke, but it's incomplete.

The important part of cloud computing is the **service model and automation around the infrastructure**.

Cloud providers give you things like:

* On-demand provisioning
* APIs
* Automated scaling
* Global infrastructure
* Managed databases
* Object storage
* Load balancers
* Monitoring
* Identity/security systems
* Infrastructure automation
* Pay-as-you-go billing

So you're not merely renting a computer.

You're accessing an **on-demand computing platform**.

---

# 13. The evolution in one picture

This is the mental model I want you to remember:

```text
PHYSICAL SERVER
      │
      │ expensive
      ▼
DEDICATED SERVER
      │
      │ share hardware
      ▼
SHARED HOSTING
      │
      │ virtualization
      ▼
VPS
      │
      │ automation + massive infrastructure
      │ + on-demand services
      ▼
CLOUD COMPUTING
      │
      ├── Compute
      ├── Storage
      ├── Databases
      ├── Networking
      ├── Security
      ├── Scaling
      ├── Monitoring
      └── Managed services
```

And that's basically the historical progression we're interested in.

---

# 14. Where AWS fits

AWS is a **cloud service provider**.

Other major providers include:

* Microsoft Azure
* Google Cloud
* AWS

AWS provides services such as:

```text
AWS
│
├── EC2       → Compute
├── S3        → Object storage
├── RDS       → Managed databases
├── VPC       → Networking
├── IAM       → Identity/security
├── ELB       → Load balancing
├── Lambda    → Serverless compute
└── CloudWatch→ Monitoring
```

And this is why your AWS course is teaching all these seemingly unrelated things.

They're actually pieces of the same system:

> **How do we build, deploy, secure, scale, store data for, and monitor applications without owning the physical infrastructure ourselves?**

That's the fundamental question cloud computing answers.

---

## One final distinction

Don't think of this as:

**Dedicated → Shared → VPS → Cloud = completely separate technologies.**

They're different **hosting/infrastructure models**, and modern cloud platforms still use technologies such as **physical servers and virtualization underneath**.

The abstraction has simply become much more powerful.

```text
You
 │
 │ "Give me a server"
 ▼
Cloud API
 │
 ▼
Virtualization / Infrastructure
 │
 ▼
Physical hardware
```

You interact primarily with the **cloud abstraction**, not the physical machine.

