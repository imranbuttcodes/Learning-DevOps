# AWS EBS — Elastic Block Store

## 1. First: What problem does EBS solve?

Remember our EC2 instance:

```text
                EC2 Instance
              ┌───────────────┐
              │               │
              │     CPU       │
              │     RAM       │
              │               │
              │   Operating   │
              │    System     │
              │               │
              └───────┬───────┘
                      │
                      ▼
                  Storage
```

Where does the operating system, files, applications, databases, etc. actually live?

**Storage.**

AWS provides **EBS — Elastic Block Store** for persistent block storage attached to EC2.

Think of EBS as:

> **A virtual hard drive/SSD that you can attach to an EC2 instance.**

---

# 2. EC2 vs EBS

This distinction is extremely important.

```text
EC2 = Compute
EBS = Storage
```

More specifically:

```text
                 AWS
                  │
          ┌───────┴───────┐
          │               │
        EC2              EBS
     Virtual Server   Virtual Disk
          │               │
       CPU/RAM          Data
          │               │
          └───────┬───────┘
                  │
              Attached
```

So when you launch an EC2 instance, you are essentially saying:

> "Give me a virtual computer."

And EBS provides the persistent disk attached to that computer.

---

# 3. Why is it called "Block" Storage?

This is a fundamental concept.

EBS is **block storage**.

Your disk is conceptually divided into fixed-size blocks:

```text
EBS Volume

┌──────┬──────┬──────┬──────┬──────┐
│Block │Block │Block │Block │Block │
│  1   │  2   │  3   │  4   │  5   │
└──────┴──────┴──────┴──────┴──────┘
```

The operating system can read/write these blocks.

That's fundamentally different from something like S3, which we'll learn separately.

### Block storage

```text
EBS
 ↓
Virtual disk
 ↓
Filesystem
 ↓
Files
```

### Object storage

```text
S3
 ↓
Objects
 ↓
Files/data + metadata
```

So:

> **EBS behaves like a disk.**

> **S3 behaves like an object storage service.**

---

# 4. What happens when you launch an EC2?

Remember when we launched our instances?

We selected something like:

```text
Root volume
8 GiB
gp3
```

That was an **EBS volume**.

For example:

```text
EC2: imranserver
       │
       │
       ▼
EBS Volume
8 GiB
gp3
       │
       ▼
Amazon Linux 2023
```

The operating system is stored on that root EBS volume.

So when your EC2 boots:

```text
EBS
 │
 ├── Amazon Linux
 ├── system files
 ├── installed packages
 ├── configuration
 └── your files
```

---

# 5. EBS Is Persistent

This is one of its biggest advantages.

Suppose:

```text
EC2
 ↓
EBS
 ↓
my files
```

You **stop** the EC2 instance.

The EBS volume remains.

Then you start the EC2 again:

```text
EC2 starts
    ↓
EBS attached
    ↓
Your data is still there
```

That's why EBS is called **persistent block storage**.

---

# 6. What happens when you TERMINATE EC2?

This is where you need to be careful.

By default, the **root EBS volume** is commonly configured with:

```text
DeleteOnTermination = true
```

So:

```text
Terminate EC2
      ↓
Root EBS volume
      ↓
Deleted
```

But EBS volumes can also be configured to survive termination.

For example:

```text
EC2
 │
 ├── Root EBS
 │      └── Delete on termination = YES
 │
 └── Data EBS
        └── Delete on termination = NO
```

Then:

```text
Terminate EC2
      │
      ├── Root volume → deleted
      │
      └── Data volume → remains
```

That's extremely useful for persistent application data.

---

# 7. EBS Volume vs EC2 Instance

Think of them as separate resources.

```text
                 EC2
          ┌───────────────┐
          │ CPU           │
          │ RAM           │
          │ OS            │
          └───────┬───────┘
                  │
             attached to
                  │
                  ▼
             ┌─────────┐
             │   EBS   │
             │ Volume  │
             └─────────┘
```

You can generally:

* create an EBS volume
* attach it to an EC2
* detach it
* attach it to another compatible EC2
* create snapshots
* resize certain EBS volumes

This separation is very important in cloud architecture.

---

# 8. Example: Database Server

Imagine we build a MongoDB server:

```text
EC2
│
├── CPU
├── RAM
│
└── EBS
     │
     └── MongoDB data
```

If the EC2 instance has a problem, you don't necessarily want your database data to disappear with the server.

You could have:

```text
EC2-1
   │
   └──── EBS Data Volume
             │
             │ detach
             ▼
           EC2-2
```

The storage and compute resources can therefore be managed somewhat independently.

---

# 9. EBS Volume Types

This is where AWS gives you different types of virtual disks.

The main families you'll encounter are:

### General Purpose SSD

```text
gp3
gp2
```

These are the most important for you initially.

`gp3` is the modern general-purpose SSD choice for many workloads.

Think:

> **Good balance of price and performance.**

---

### Provisioned IOPS SSD

```text
io2
```

These are designed for workloads requiring very high and predictable I/O performance.

Think:

```text
High-performance database
        ↓
Very demanding I/O
        ↓
Provisioned IOPS
```

---

### Throughput Optimized HDD

```text
st1
```

Designed for workloads where **large sequential throughput** matters more than ultra-low latency.

---

### Cold HDD

```text
sc1
```

Designed for less frequently accessed data where lower storage cost is important.

---

### Simple mental model

```text
                 EBS
                  │
       ┌──────────┴──────────┐
       │                     │
      SSD                   HDD
       │                     │
   ┌───┴────┐           ┌────┴────┐
   │        │           │         │
  gp3      io2         st1       sc1
General   High IOPS   Throughput  Cold
purpose
```

For your DevOps work:

> **Know `gp3` very well. Know what `io2`, `st1`, and `sc1` are conceptually.**

You don't need to memorize every specification right now.

---

# 10. What is `gp3`?

You saw this when creating your EC2:

```text
8 GiB
gp3
```

`gp3` means:

```text
General Purpose SSD
```

It's designed to provide a good balance between:

* cost
* performance
* flexibility

And importantly, with `gp3`, storage capacity and performance characteristics can be provisioned more independently than with older `gp2` behavior.

For our normal:

```text
FastAPI
Docker
Node.js
Nginx
small database
development server
```

workloads, **gp3 is usually the kind of EBS volume you'll encounter.**

---

# 11. EBS Is AZ-Specific

This connects directly to what we just learned about **Regions and Availability Zones**.

An EBS volume exists in a particular **Availability Zone**.

For example:

```text
Region: us-east-1
│
├── AZ-A
│    └── EBS Volume A
│
├── AZ-B
│    └── EBS Volume B
│
└── AZ-C
     └── EBS Volume C
```

You generally can't simply attach:

```text
EBS in us-east-1a
```

directly to:

```text
EC2 in us-east-1b
```

because they're in different Availability Zones.

This is an important relationship:

```text
Region
   │
   ├── AZ-a
   │    ├── EC2
   │    └── EBS
   │
   └── AZ-b
        ├── EC2
        └── EBS
```

---

# 12. EBS vs Instance Store

You'll eventually encounter another storage type:

**Instance Store.**

Don't confuse it with EBS.

### EBS

```text
Persistent
      ↓
EC2 can stop
      ↓
Data remains
```

### Instance Store

```text
Physically attached to host
      ↓
Very fast
      ↓
Temporary/ephemeral
      ↓
Data can be lost when instance is stopped/terminated/reallocated
```

So:

```text
EBS
→ Persistent storage

Instance Store
→ Temporary local storage
```

For normal application/database persistence, EBS is much more important.

---

# 13. EBS Snapshots

This is another **very important DevOps concept**.

You can create a snapshot of an EBS volume.

Think:

```text
EBS Volume
     │
     │ Snapshot
     ▼
┌─────────────┐
│   Backup    │
└─────────────┘
```

For example:

```text
EBS
│
├── OS
├── application
└── database data
       │
       ▼
    Snapshot
```

If something goes wrong, snapshots can be used as a basis for creating/restoring volumes.

This is conceptually similar to:

> **"Take a backup/checkpoint of this disk."**

---

# 14. EBS vs S3

This distinction will matter a LOT in AWS.

|                | EBS                            | S3                               |
| -------------- | ------------------------------ | -------------------------------- |
| Type           | Block storage                  | Object storage                   |
| Attached to    | EC2                            | Not attached like a disk         |
| Looks like     | Hard drive                     | Object repository                |
| Use            | OS, apps, databases            | Files, images, backups, datasets |
| Persistent     | Yes                            | Yes                              |
| Filesystem     | Yes, after formatting/mounting | No traditional filesystem        |
| Typical access | Disk I/O                       | API/HTTP                         |

Mental model:

```text
"I need a disk for my server"
             ↓
            EBS


"I need to store files/data in AWS"
             ↓
             S3
```

---

# 15. The Full EC2 + EBS Picture

Now combine everything we've learned:

```text
                         AWS Region
                            │
                     Availability Zone
                            │
             ┌──────────────┴──────────────┐
             │                             │
          EC2 Instance                 EBS Volume
             │                             │
       ┌─────┴─────┐                  ┌────┴────┐
       │           │                  │         │
      CPU         RAM               Filesystem  Data
       │           │
       └─────┬─────┘
             │
             └────────── attached ──────────┘
```

And remember:

```text
EC2 = compute
EBS = persistent block storage
S3 = object storage
```

That's the core mental model.

---

# 16. What You Actually Need to Know for Your DevOps Path

Don't try to memorize the entire EBS documentation.

For now, master these:

### Must know

* What EBS is
* Block storage
* EBS volume
* Root volume
* Persistent storage
* EBS vs instance store
* EBS vs S3
* EBS is AZ-specific
* `gp3`
* EBS snapshots
* What happens to EBS when EC2 is stopped/terminated
* `DeleteOnTermination`

### Later

* IOPS
* Throughput
* Provisioned IOPS
* EBS optimization
* Multi-Attach
* encryption
* resizing volumes
* snapshot lifecycle/automation

The next natural step after this is **hands-on EBS**: create a separate `gp3` volume → attach it to an EC2 → format it → mount it → write data → detach it → attach it to another instance. That's where EBS will really click.
