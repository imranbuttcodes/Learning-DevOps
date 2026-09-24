### 🧠 AMI vs Launch Template

Think of them as two different blueprints:

```text
              AMI                         Launch Template
        "WHAT is inside?"              "HOW should it launch?"
              │                               │
              ↓                               ↓
       ┌──────────────┐               ┌──────────────────┐
       │ OS            │               │ AMI ID           │
       │ Installed     │               │ Instance type    │
       │ software      │               │ VPC/subnet       │
       │ Files         │               │ Security groups  │
       │ Configuration │               │ Key pair         │
       │ Data*         │               │ EBS configuration│
       └──────────────┘               │ User data        │
                                      └──────────────────┘
```

### AMI

An **AMI captures the machine image** — particularly the OS and the contents/configuration of its root volume(s).

For example:

```text
EC2
 ├── Amazon Linux
 ├── Apache installed
 ├── Python installed
 ├── /var/www/html/
 └── application files
```

You can create an AMI from that and later launch another EC2 from it.

But the AMI **doesn't mean**:

> "Put this machine in VPC X, subnet Y, with Security Group Z."

Those are launch/network configuration decisions.

---

### Launch Template

A Launch Template describes **how you want the EC2 to be launched**.

For example:

```text
Launch Template
│
├── AMI → ami-xxxxx
├── Instance type → t3.micro
├── Key pair → my-key
├── Network
│    ├── VPC → VPC-A
│    └── Subnet → subnet-123
├── Security Group → web-sg
├── EBS configuration
└── User Data
```

So you could have:

```text
              Launch Template
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       EC2         EC2         EC2
        │           │           │
     Same AMI    Same AMI    Same AMI
     Same type   Same type   Same type
     Same SG     Same SG     Same SG
     etc...
```

That's **exactly why Launch Templates are so useful with ASGs**.

### One correction to your wording

You said:

> "we use image to copy the EC2 itself"

That's a good beginner mental model, but technically:

**AMI = image of the machine's software/storage state, not the entire EC2 resource.**

The EC2 instance itself also has things like:

* networking
* subnet
* security groups
* instance type
* IAM instance profile
* key pair
* EBS attachment configuration

Those can be specified separately when launching.

So the clean mental model is:

> 🟦 **AMI = what the server contains**
> 🟨 **Launch Template = how the server should be launched**
> 🟩 **ASG = how many servers should exist and when to replace/scale them**

And that trio is the foundation of the architecture we're building now.
