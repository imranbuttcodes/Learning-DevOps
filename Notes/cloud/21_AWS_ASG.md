**ASG (Auto Scaling Group)** is the natural next step after what we just learned with the ALB.

You already have:

```text
Internet
    ↓
   ALB
    ↓
Target Group
    ↓
EC2 #1
EC2 #2
EC2 #3
```

ASG adds **automatic management of those EC2 instances**.

# 🚀 What is an ASG?

**Auto Scaling Group (ASG)** is an AWS service that automatically manages a group of EC2 instances according to rules you define.

The basic idea:

> **"Keep the right number of EC2 servers running, and automatically replace or add/remove servers when necessary."**

For example:

```text
ASG desired capacity = 3

        ASG
         │
   ┌─────┼─────┐
   ↓     ↓     ↓
 EC2   EC2   EC2
  #1    #2    #3
```

If EC2 #2 crashes:

```text
EC2 #1 ✅
EC2 #2 ❌
EC2 #3 ✅
```

ASG notices that only **2** instances are running.

It can launch:

```text
EC2 #4 ✅
```

Now:

```text
EC2 #1
EC2 #3
EC2 #4
```

Back to the desired capacity of 3.

---

# 🧠 Why do we need ASG?

Imagine you're running an API.

During normal traffic:

```text
100 requests/min

EC2 #1
EC2 #2
```

Two servers are enough.

Then traffic suddenly increases:

```text
10,000 requests/min
```

Two EC2s might struggle.

You could manually launch more:

```text
EC2 #1
EC2 #2
EC2 #3
EC2 #4
EC2 #5
```

But that's not what we want.

Instead:

```text
Traffic increases
       ↓
ASG detects condition
       ↓
Launch more EC2s
       ↓
ALB distributes traffic
```

When traffic drops:

```text
Traffic decreases
       ↓
ASG detects condition
       ↓
Terminate unnecessary EC2s
       ↓
Save resources/cost
```

That's **elasticity** in practice.

---

# 🔥 ASG has 3 important numbers

This is extremely important.

Suppose we configure:

```text
Minimum capacity = 2
Desired capacity = 3
Maximum capacity = 6
```

Think of them like this:

### Minimum

> "Never intentionally go below 2."

```text
MIN = 2
```

### Desired

> "Normally, I want 3."

```text
DESIRED = 3
```

### Maximum

> "Never intentionally scale beyond 6."

```text
MAX = 6
```

So:

```text
2 ≤ EC2 instances ≤ 6

Normal:
        3 instances

Heavy traffic:
        4 → 5 → 6

Low traffic:
        3 → 2
```

---

# ⚡ Scaling

There are two major directions.

### Scale OUT

Add instances:

```text
2 EC2
  ↓
3 EC2
  ↓
4 EC2
```

This is **horizontal scaling**.

### Scale IN

Remove instances:

```text
5 EC2
  ↓
4 EC2
  ↓
3 EC2
```

---

# 🏗️ But there's a problem...

ASG needs to know:

> **"What kind of EC2 should I launch?"**

You don't want AWS randomly creating some EC2.

For example:

```text
ASG
 ↓
???
 ↓
t3.micro?
Amazon Linux?
Which AMI?
Which security group?
Which key?
Which subnet?
```

That's where the **Launch Template** comes in.

---

# 🎯 Launch Template

You can think of a Launch Template as the **recipe for creating an EC2 instance**.

For example:

```text
Launch Template
├── AMI: Amazon Linux 2023
├── Instance type: t3.micro
├── Key pair: my-key
├── Security Group: web-server-sg
├── EBS: 8 GB gp3
└── User Data: install Apache
```

Then:

```text
                 Launch Template
                       │
            ┌──────────┼──────────┐
            ↓          ↓          ↓
           EC2        EC2        EC2
            #1         #2         #3
```

So:

**Launch Template = instructions for creating an EC2.**

**ASG = manages how many EC2s should exist.**

---

# 🔥 Now combine everything we've learned

This is where AWS starts becoming a real architecture rather than individual services.

```text
                         Internet
                            │
                            ↓
                         Route 53
                            │
                            ↓
                          ALB
                            │
                     Target Group
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           EC2 #1         EC2 #2        EC2 #3
              ↑             ↑             ↑
              └─────────────┼─────────────┘
                            │
                           ASG
                            │
                     Launch Template
```

ASG creates/manages the EC2s.

ALB sends traffic to them.

Target Group tracks their health.

Launch Template tells ASG how to create them.

---

# 🧩 ASG + ALB = powerful combination

Suppose you currently have:

```text
ASG
Desired = 3

EC2 #1 ✅
EC2 #2 ✅
EC2 #3 ✅

       ↓

ALB
```

Now EC2 #2 crashes.

ASG launches:

```text
EC2 #4
```

But there's another important piece:

**ASG can automatically register newly launched instances with the Target Group.**

So you don't have to manually go:

> Target Group → Register targets → select EC2 #4

Instead:

```text
ASG launches EC2 #4
          ↓
EC2 #4 joins Target Group
          ↓
ALB health check
          ↓
Healthy ✅
          ↓
ALB starts sending traffic
```

That's the architecture we were missing when you manually added your third instance earlier.

---

# 📈 How does ASG know when to scale?

You can configure **scaling policies**.

For example:

```text
Average CPU > 70%
        ↓
Launch another EC2
```

Or:

```text
Average CPU < 30%
        ↓
Remove an EC2
```

There are several scaling approaches, including:

* **Target tracking** — "Keep average CPU around 50%."
* **Step scaling** — scale by different amounts depending on how far the metric crosses a threshold.
* **Scheduled scaling** — scale at known times.
* **Predictive scaling** — AWS uses historical patterns to anticipate demand.

For now, **target tracking** is the most important one to understand.

---

# 🧠 One subtle but VERY important distinction

Don't think:

> "ASG automatically makes my application faster."

It doesn't.

ASG's job is primarily:

> **manage the number of EC2 instances.**

ALB's job:

> **distribute traffic.**

Target Group's job:

> **organize targets + health checks.**

Launch Template's job:

> **define how new EC2 instances are created.**

So:

```text
Launch Template
       ↓
     ASG
       ↓
 EC2 EC2 EC2
   ↓   ↓   ↓
 Target Group
       ↓
      ALB
       ↓
    Users
```

That's the mental model I want you to have before we touch the AWS console.

## Next hands-on

We'll build exactly this:

```text
Launch Template
      ↓
ASG
 ┌────┼────┐
EC2  EC2  EC2
 └────┼────┘
      ↓
Target Group
      ↓
     ALB
```