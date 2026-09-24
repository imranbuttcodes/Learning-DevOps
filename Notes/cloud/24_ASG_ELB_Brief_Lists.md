**Brief ELB + ASG recap**:

### ⚖️ Elastic Load Balancing (ELB)

* **ELB** = distributes incoming traffic across backend servers.
* **ALB (Application Load Balancer)** → HTTP/HTTPS, Layer 7.
* ALB does **not run the application**; EC2 instances do.
* **Target Group** = group of backend targets that ALB sends traffic to.
* **Health checks** → ALB checks whether targets are healthy before routing traffic.
* **Security Group** → controls who can connect to the ALB/EC2.
* **Load-balancing algorithm**:

  * Round Robin — default
  * Least Outstanding Requests (LOR) — alternative
* ALB is **Regional** and can use multiple AZs.
* Users normally access an ALB through its **DNS name**, not a fixed IP.
* Typical architecture:

```text
User
  ↓
Route 53
  ↓
ALB
  ↓
Target Group
  ↓
EC2 #1
EC2 #2
EC2 #3
```

---

### 📈 Auto Scaling Group (ASG)

* **ASG** = manages a fleet of EC2 instances.
* **Desired capacity** → number of instances ASG currently wants.
* **Minimum capacity** → lowest number it should maintain.
* **Maximum capacity** → highest number it can scale to.
* ASG can:

  * Launch new EC2s
  * Terminate instances during scale-in
  * Replace unhealthy instances
  * Scale based on policies/metrics
* ASG can work **without ALB**.
* When connected to an ALB, ASG automatically registers its instances with the **Target Group**.
* ASG can distribute instances across multiple **Availability Zones**.
* **Deregister ≠ terminate**:

  * Deregister → remove from traffic
  * Terminate → delete the EC2 instance

### 🔗 How ALB + ASG work together

```text
                 ALB
                  ↓
            Target Group
             ↙    ↓    ↘
          EC2   EC2   EC2
           ↑     ↑     ↑
              ASG
                ↓
        "How many instances?"
```

### 🧠 The key mental model

**ALB:** *“Where should this request go?”*
**Target Group:** *“Which servers can receive it?”*
**ASG:** *“How many servers should exist?”*
**Launch Template:** *“What should each new server look like?”*
**AMI:** *“What is inside that server?”*
