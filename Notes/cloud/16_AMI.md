# AMI — Amazon Machine Image

Before touching the console, let's build the mental model first.

## 1. What is an AMI?

**AMI = Amazon Machine Image.**

An AMI is a **template/image used to launch EC2 instances**.

Think of it as a **blueprint for an EC2 machine**.

```text
                    AMI
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
       Launch                 Launch
       EC2 #1                 EC2 #2
          │                     │
          ↓                     ↓
      Same base              Same base
      configuration           configuration
```

Instead of manually setting up every new EC2 server from scratch, you can create an AMI from an already-configured instance and use it to launch new instances.

---

# 2. What does an AMI contain?

An AMI contains the information needed to launch an EC2 instance, including the **root volume's image/content** and the configuration needed to launch it.

For example, imagine you configure an EC2 server:

```text
Amazon Linux
      +
Python
      +
FastAPI
      +
Docker
      +
Your application
      +
Configuration
```

You can create an AMI from that instance.

Then:

```text
Configured EC2
      │
      │ Create AMI
      ↓
     AMI
      │
      ├──────────→ New EC2
      │
      ├──────────→ New EC2
      │
      └──────────→ New EC2
```

The new instances can start from that prepared image rather than beginning with a completely clean machine.

---

# 3. Why do we need AMIs?

Imagine you're deploying your backend.

Without an AMI:

```text
Launch EC2
   ↓
Install Linux/configure
   ↓
Install Docker
   ↓
Install dependencies
   ↓
Copy application
   ↓
Configure everything
   ↓
Ready
```

Do that 10 times and it's painful.

With an AMI:

```text
                    AMI
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        EC2 #1     EC2 #2     EC2 #3
          │          │          │
       Ready      Ready      Ready
```

That's one of the major reasons AMIs are useful.

---

# 4. AMI vs EBS Snapshot

This is **very important** because you just learned snapshots.

They're related, but they're **not the same thing**.

### EBS Snapshot

A snapshot is a backup of an **EBS volume**.

```text
EBS Volume
    ↓
Snapshot
```

### AMI

An AMI is a **launch template/image for EC2**.

```text
AMI
 ↓
Launch EC2
```

Think:

> **Snapshot = backup of a disk**

> **AMI = image/template for creating an EC2 machine**

---

# 5. How are they related?

An AMI can reference one or more EBS snapshots for the instance's EBS-backed volumes.

Conceptually:

```text
              AMI
               │
       ┌───────┴────────┐
       ↓                ↓
Root-volume          Additional
snapshot             volume info
       │
       ↓
New EC2
       │
       ↓
New EBS volume
```

So there's a connection:

```text
EBS Volume
    ↓
Snapshot
    ↓
AMI
    ↓
New EC2
```

But don't think:

> "AMI = snapshot"

They serve different purposes.

---

# 6. Example

Suppose you launch Amazon Linux 2023:

```text
EC2
├── Amazon Linux
├── Docker
├── Python
├── FastAPI
└── my-api
```

You configure everything exactly how you want.

Then:

```text
EC2
 ↓
Create AMI
 ↓
my-fastapi-server-v1
```

Now you can launch:

```text
my-fastapi-server-v1
        │
        ├── EC2 #1
        ├── EC2 #2
        ├── EC2 #3
        └── EC2 #4
```

Each starts from that image.

---

# 7. AMI and Scaling

This becomes especially useful with **Auto Scaling**.

Imagine traffic suddenly increases:

```text
Normal traffic

EC2 #1
```

Traffic increases:

```text
EC2 #1
   +
EC2 #2
   +
EC2 #3
```

Where do #2 and #3 come from?

An Auto Scaling Group can launch new instances using an **AMI**.

```text
                 AMI
                  │
                  ↓
           Auto Scaling Group
             │      │      │
             ↓      ↓      ↓
           EC2 #1 EC2 #2 EC2 #3
```

This is one of the most important real-world uses of AMIs.

---

# 8. Public AMIs vs Your Own AMIs

You don't have to create every AMI yourself.

AWS provides public AMIs.

For example:

```text
AWS
 │
 ├── Amazon Linux AMI
 ├── Ubuntu AMI
 └── Other OS images
```

When you launched your Amazon Linux 2023 EC2 earlier, you selected an **Amazon Linux AMI**.

You can also create your own:

```text
AWS-provided AMI
       ↓
Launch EC2
       ↓
Customize everything
       ↓
Create your own AMI
```

---

# 9. AMI IDs

Every AMI has an ID.

Example:

```text
ami-0fef201115eefe936
```

That's the AMI ID for one of the Amazon Linux images you encountered earlier.

When launching an EC2 instance, AWS essentially needs to know:

```text
Which AMI should I use?
```

Then it creates the instance based on that image.

---

# 10. AMIs are Region-specific

This is an important AWS concept.

An AMI is associated with a specific **Region**.

For example:

```text
AMI
us-east-1
```

doesn't automatically mean you can use that exact AMI ID in:

```text
eu-west-1
```

You can **copy an AMI to another Region**.

```text
us-east-1
   │
   │ Copy AMI
   ↓
eu-west-1
```

Then you can launch EC2 instances from the copied AMI in that Region.

---

# 11. AMI vs Snapshot vs EBS

Here's the hierarchy I want you to remember:

```text
EBS Volume
     │
     │ backup
     ↓
  Snapshot
```

Whereas:

```text
AMI
  │
  │ launch
  ↓
EC2 Instance
  │
  ↓
EBS Volumes
```

And an AMI for an EBS-backed instance can be based on snapshots of the source instance's EBS volumes.

### Simple analogy

Imagine a computer:

```text
EBS Volume = Hard drive
Snapshot   = Backup of hard drive
AMI        = Complete machine blueprint
EC2        = Actual running computer
```

That's the mental model.

---

## 12. One more important distinction

An AMI is **not a running server**.

```text
AMI
 ↓
Blueprint/template
```

EC2 is the actual running compute:

```text
AMI
 ↓
Launch
 ↓
EC2
 ↓
Running application
```

So if someone says:

> "Launch an EC2 from this AMI"

they mean:

> "Use this machine image/template as the starting point for the new EC2 instance."

---

