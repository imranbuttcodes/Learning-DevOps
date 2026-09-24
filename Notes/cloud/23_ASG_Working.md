# 🔄 How Auto Scaling works

Imagine you configure:

```text
Minimum capacity = 2
Desired capacity = 2
Maximum capacity = 5
```

And your Launch Template says:

```text
AMI = your Apache-server AMI
Instance type = t3.micro
Security Group = web-server-sg
```

Then your ASG maintains the fleet:

```text
                 ASG
                  │
          ┌───────┴───────┐
          ↓               ↓
        EC2 #1           EC2 #2
        Healthy          Healthy
```

## 1. ASG continuously monitors the group

ASG keeps track of things such as:

* How many instances are running
* Whether instances are healthy
* Scaling policies
* Desired/min/max capacity
* Metrics used for scaling

It doesn't simply ask:

> "Is the CPU high?"

It follows the **scaling policy you configured**.

---

# 📈 2. Traffic increases

Suppose your application gets much busier.

You configure a target-tracking policy such as:

> **Keep average CPU utilization around 50%.**

Initially:

```text
EC2 #1 → 45% CPU
EC2 #2 → 55% CPU

Average ≈ 50%
```

Everything is fine.

Then traffic increases:

```text
EC2 #1 → 85%
EC2 #2 → 90%

Average ≈ 87.5%
```

The scaling policy detects that the group is above its target.

---

# ➕ 3. ASG scales OUT

ASG decides that more capacity is needed.

It uses your **Launch Template**:

```text
ASG
 │
 │ "I need another server."
 ↓
Launch Template
 │
 │ AMI
 │ Instance type
 │ Security Group
 │ etc.
 ↓
New EC2
```

Now:

```text
Before:

EC2 #1
EC2 #2


After:

EC2 #1
EC2 #2
EC2 #3
```

If necessary, it can continue:

```text
2 → 3 → 4 → 5
```

but never beyond your configured maximum of 5.

---

# ❤️ 4. New instance becomes healthy

This is where your **ALB + Target Group** comes in.

The new EC2 starts:

```text
EC2 #3
   ↓
Boot
   ↓
Apache starts
   ↓
Target Group health check
   ↓
Healthy ✅
```

Once healthy, the ALB can send requests to it.

So:

```text
                 ALB
                  │
             Target Group
          ┌───────┼───────┐
          ↓       ↓       ↓
        EC2 #1  EC2 #2  EC2 #3
```

Now the workload is distributed across more servers.

---

# 📉 5. Traffic decreases

Later, traffic drops.

```text
EC2 #1 → 20%
EC2 #2 → 25%
EC2 #3 → 15%
```

The scaling policy determines that the group has more capacity than needed.

ASG can **scale in**:

```text
3 EC2
  ↓
2 EC2
```

But it won't go below:

```text
Minimum capacity = 2
```

So your fleet settles back at 2.

---

# 💥 6. What if an EC2 crashes?

This is slightly different from scaling based on CPU.

Suppose:

```text
EC2 #1 ✅
EC2 #2 ❌
EC2 #3 ✅
```

ASG's **health checking** detects that an instance is unhealthy.

If your desired capacity is 3:

```text
Desired = 3

Healthy instances = 2
```

ASG launches a replacement:

```text
Launch Template
       ↓
    EC2 #4
       ↓
Health check
       ↓
Healthy ✅
```

Now:

```text
EC2 #1 ✅
EC2 #3 ✅
EC2 #4 ✅
```

That's called **self-healing**.

---

# 🔥 The complete picture

This is the architecture you should memorize conceptually:

```text
                         USERS
                           │
                           ↓
                          ALB
                           │
                           ↓
                     Target Group
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          EC2 #1        EC2 #2        EC2 #3
             ↑             ↑             ↑
             └─────────────┼─────────────┘
                           │
                          ASG
                           │
                    Scaling Policy
                           │
                    Launch Template
                           │
                           ↓
                         AMI
```

Each component has a specific job:

| Component           | Job                                      |
| ------------------- | ---------------------------------------- |
| **AMI**             | Defines what's inside the server         |
| **Launch Template** | Defines how to launch the server         |
| **ASG**             | Maintains the desired number of servers  |
| **Scaling Policy**  | Determines when/how to change capacity   |
| **Target Group**    | Tracks backend instances + health checks |
| **ALB**             | Distributes incoming requests            |

### The key distinction

**ASG doesn't directly respond to "traffic."**

It responds to **metrics and scaling policies**.

For example:

```text
CloudWatch metric
       ↓
Scaling Policy
       ↓
ASG
       ↓
Launch / terminate EC2
```

And that's why **CloudWatch** is closely related to Auto Scaling.

---

## One more important thing: ASG ≠ automatically "CPU > X"

You choose the scaling strategy.

For example:

**Target tracking**

> Keep average CPU around 50%.

**Step scaling**

> If CPU > 70%, add 1.
> If CPU > 90%, add 2.

**Scheduled scaling**

> Every weekday at 9 AM, increase desired capacity to 5.

So ASG is basically the **controller**, while the scaling policy tells that controller **what behavior you want**.

For our hands-on, we'll use **Target Tracking** because it's the easiest way to see the entire system working.
