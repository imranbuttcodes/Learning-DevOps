**This is the point where all the AWS pieces finally come together.** Let's build one realistic system from **your code → GitHub → CI/CD → Docker → AWS → users** and connect every concept we've learned.

We'll use a **FastAPI AI backend** as the example because that's directly relevant to what you're building.

---

# 🏗️ The Complete Production System

Imagine you have:

```text
FastAPI + AI backend
```

with endpoints like:

```text
POST /chat
GET  /health
POST /documents
```

Users access it from a frontend/mobile app.

The production architecture could look like:

```text
                         🌍 USERS
                            │
                            │ HTTPS
                            ↓
                    ┌─────────────────┐
                    │   Route 53      │
                    │      DNS        │
                    └────────┬────────┘
                             │
                             ↓
                    ┌─────────────────┐
                    │   CloudFront    │
                    │      CDN        │
                    └────────┬────────┘
                             │
                             ↓
                    ┌─────────────────┐
                    │       ALB       │
                    │ Load Balancer   │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    ↓                 ↓
              ┌──────────┐      ┌──────────┐
              │   EC2    │      │   EC2    │
              │ FastAPI  │      │ FastAPI  │
              └────┬─────┘      └────┬─────┘
                   │                  │
                  EBS                EBS
                   │                  │
                   └────────┬─────────┘
                            │
                            ↓
                    ┌───────────────┐
                    │      RDS      │
                    │  PostgreSQL   │
                    └───────────────┘
```

And around all of this:

```text
VPC
├── Subnets
├── Security Groups
├── Route tables
└── Internet connectivity
```

And AWS manages access through:

```text
IAM
```

And deployment through:

```text
GitHub Actions
      ↓
Docker
      ↓
ECR
      ↓
EC2
```

Let's go from the beginning.

---

# 1. 👨‍💻 You write the application

You have something like:

```text
project/
│
├── app/
│   ├── main.py
│   ├── routes/
│   └── services/
│
├── tests/
│
├── Dockerfile
├── requirements.txt
└── README.md
```

Your FastAPI application:

```text
Internet request
       ↓
   FastAPI
       ↓
 business logic
       ↓
 database / AI service
```

At this point, **nothing is AWS-specific yet.**

It's just your application.

---

# 2. 📦 Docker packages your application

You create a Dockerfile:

```text
Dockerfile
    ↓
Docker build
    ↓
Docker Image
    ↓
Container
```

The image contains things your application needs:

```text
Docker Image
├── Linux userspace
├── Python
├── dependencies
├── FastAPI
└── your application
```

Then:

```text
docker run
```

creates a container from that image.

You already practiced this with your Docker test application.

---

# 3. 🧪 GitHub Actions becomes your CI/CD pipeline

You push:

```text
git push
     ↓
GitHub
     ↓
GitHub Actions
```

Your workflow might do:

```text
Checkout code
     ↓
Install dependencies
     ↓
Run tests
     ↓
Build Docker image
     ↓
Push image
```

For example:

```text
GitHub
   │
   ↓
GitHub Actions
   │
   ├── pytest
   │
   ├── docker build
   │
   └── docker push
```

Now you have a tested container image.

---

# 4. 📦 Where does the Docker image go?

In AWS, you'd commonly use:

**Amazon ECR — Elastic Container Registry**

Think:

```text
Docker Hub
      ↕
     ECR
```

ECR is AWS's container image registry.

So:

```text
GitHub Actions
      │
      │ docker push
      ↓
     ECR
      │
      │ docker pull
      ↓
     EC2
```

Your EC2 can pull your application image from ECR.

---

# 5. 🖥️ Now we need COMPUTE

This is where **EC2** enters.

EC2 is your virtual server.

```text
EC2
│
├── CPU
├── RAM
├── Operating System
└── Network interface
```

We learned instance types here:

```text
t3.micro
t3.small
m7i.large
c7i.large
...
```

For example:

```text
t3.micro
↓
2 vCPU
1 GiB RAM
```

The instance type determines the compute resources available to your server.

---

# 6. 💾 Where does the EC2's disk come from?

**EBS.**

Your EC2 gets a root EBS volume:

```text
EC2
│
└── EBS
     │
     ├── Linux
     ├── Docker
     ├── configuration
     └── files
```

Remember:

> EC2 = computer
> EBS = its persistent disk

Your `8 GiB gp3` root volume from the AWS console was exactly this.

---

# 7. 🏭 But how do we create identical servers?

Now we use the **AMI**.

Suppose you prepare a server:

```text
EC2
│
├── Amazon Linux
├── Docker
├── monitoring agent
├── configurations
└── application setup
```

You create:

```text
AMI
```

The AMI can contain the image of the relevant EBS-backed volumes.

Now:

```text
              AMI
               │
       ┌───────┼───────┐
       ↓       ↓       ↓
     EC2 #1  EC2 #2  EC2 #3
```

So instead of manually configuring every server, you have a standardized starting image.

---

# 8. 📋 How do we tell AWS HOW to launch it?

**Launch Template.**

Your Launch Template might say:

```text
AMI              → Webserver-v1
Instance type    → t3.micro
Key pair         → mywebserver-key
Security groups  → launch-wizard-6
Subnet           → subnet-xxxx
EBS              → 8 GiB gp3
IAM role         → EC2-role
```

So:

```text
AMI
"What should the machine contain?"

        +

Launch Template
"How should AWS launch it?"
```

Together:

```text
AMI + Launch Template
          ↓
        EC2
```

---

# 9. 📈 Now we want SCALING

Suppose your application suddenly gets:

```text
10 users
   ↓
100 users
   ↓
1,000 users
   ↓
10,000 users
```

One EC2 may not be enough.

So we use:

**Auto Scaling Group (ASG)**

```text
             Auto Scaling Group
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        EC2 #1     EC2 #2     EC2 #3
```

The ASG can maintain the desired number of instances and scale based on configured policies/metrics.

And how does it know **how to create a new EC2?**

```text
ASG
 ↓
Launch Template
 ↓
AMI
 ↓
New EC2
```

That's the connection.

---

# 10. ⚖️ But how do users reach those EC2s?

We don't want users to manually connect to:

```text
EC2 #1
EC2 #2
EC2 #3
```

Instead:

**Application Load Balancer (ALB)**

```text
                    Users
                      │
                      ↓
                     ALB
                /     |     \
               ↓      ↓      ↓
             EC2 #1 EC2 #2 EC2 #3
```

The ALB distributes requests among healthy servers.

For example:

```text
Request 1 → EC2 #1
Request 2 → EC2 #2
Request 3 → EC2 #3
Request 4 → EC2 #1
```

It also performs health checks.

If:

```text
EC2 #2 💀
```

the ALB can stop sending traffic to it.

---

# 11. 🌎 Where are these EC2s actually located?

Inside your **VPC**.

Remember:

```text
AWS Region
   │
   ├── AZ-a
   │     └── Subnets
   │
   ├── AZ-b
   │     └── Subnets
   │
   └── AZ-c
         └── Subnets
```

For example:

```text
us-east-1
│
├── AZ-a
│    └── Public/Private Subnets
│
├── AZ-b
│    └── Public/Private Subnets
│
└── AZ-c
     └── Public/Private Subnets
```

Your **VPC** might be:

```text
172.31.0.0/16
```

Then individual subnets carve out smaller networks:

```text
VPC
172.31.0.0/16
│
├── subnet A
│   172.31.1.0/24
│
├── subnet B
│   172.31.2.0/24
│
└── subnet C
    172.31.3.0/24
```

Each subnet belongs to **one AZ**.

---

# 12. 🔐 Security Groups protect the servers

You don't want:

```text
Internet → EC2 → EVERYTHING ALLOWED
```

Instead:

```text
Internet
   │
   ↓
 ALB
   │
   ↓
EC2
```

Security Groups control allowed network traffic.

For example:

```text
ALB Security Group
        │
        └── Allow HTTPS 443 from Internet

EC2 Security Group
        │
        └── Allow application port only from ALB
```

So the EC2 doesn't need to expose the application directly to the whole Internet.

---

# 13. 🔑 IAM controls AWS permissions

Suppose your FastAPI application needs to read files from S3.

**Do NOT put AWS access keys inside your code.**

Instead:

```text
EC2
 │
 ↓
IAM Role
 │
 ↓
Permissions
 │
 ↓
S3
```

IAM answers:

> **Who can do what to which AWS resource?**

For example:

```text
EC2 Role
    ↓
Allow:
s3:GetObject
    ↓
bucket/my-files/*
```

That's the least-privilege approach.

---

# 14. 🗄️ Where does our database go?

You could install PostgreSQL directly on EC2.

But normally you'd use:

**Amazon RDS**

Architecture:

```text
             EC2
              │
              │ SQL connection
              ↓
             RDS
              │
       ┌──────┴──────┐
       ↓             ↓
   PostgreSQL      Storage
```

So:

```text
EC2 = application server

RDS = database
```

Your FastAPI code might conceptually do:

```text
POST /users
      ↓
FastAPI
      ↓
SQL query
      ↓
RDS PostgreSQL
      ↓
Database result
      ↓
HTTP response
```

---

# 15. 📦 What about S3?

S3 is **object storage**.

Perfect for things like:

```text
PDFs
images
videos
documents
backups
datasets
static assets
```

Instead of storing a user's uploaded PDF on EC2's EBS:

```text
User
 ↓
FastAPI
 ↓
S3
 ↓
document.pdf
```

Then your database might store metadata:

```text
RDS

document_id
user_id
filename
s3_key
created_at
```

while the actual file lives in:

```text
S3
└── documents/
      └── abc123.pdf
```

That's a very common architecture.

---

# 16. 🌐 How does the user get to our application?

We use **Route 53** for DNS.

Suppose your domain is:

```text
api.example.com
```

User:

```text
https://api.example.com
```

DNS:

```text
api.example.com
       ↓
   Route 53
       ↓
      ALB
       ↓
      EC2
```

Route 53 essentially answers:

> "Where should this domain name go?"

---

# 17. 🚀 Where does CloudFront fit?

CloudFront is a **CDN**.

Instead of every static resource traveling all the way from your origin server:

```text
User in Pakistan
       ↓
     Origin
       ↓
    USA AWS
```

CloudFront can cache content at edge locations closer to users.

Conceptually:

```text
             CloudFront
            /     |     \
           ↓      ↓      ↓
        Edge    Edge    Edge
         ↓       ↓       ↓
       Users   Users   Users
```

For APIs, CloudFront can also sit in front of an origin depending on architecture, but its biggest conceptual role is **content delivery/caching at the edge**.

---

# 18. 📊 What about CloudWatch?

Now we need observability.

```text
EC2
 │
 ├── CPU
 ├── memory-related metrics/agents
 ├── application logs
 └── system metrics
       │
       ↓
   CloudWatch
```

CloudWatch lets you monitor things such as:

```text
CPU utilization
Request metrics
Logs
Alarms
```

For example:

```text
CPU > 70%
     ↓
CloudWatch metric/alarm
     ↓
Auto Scaling policy
     ↓
Launch additional EC2
```

Now your monitoring and scaling can work together.

---

# 19. 🔄 Now let's connect EVERYTHING

Here's the full picture:

```text
                           👤 USER
                              │
                              │ HTTPS
                              ↓
                        ┌───────────┐
                        │ Route 53  │
                        │   DNS     │
                        └─────┬─────┘
                              │
                              ↓
                        ┌───────────┐
                        │ CloudFront│
                        │    CDN    │
                        └─────┬─────┘
                              │
                              ↓
                    ┌──────────────────┐
                    │       ALB        │
                    │ Load Balancer    │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              ↓              ↓              ↓
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │   EC2    │   │   EC2    │   │   EC2    │
        │ FastAPI  │   │ FastAPI  │   │ FastAPI  │
        └────┬─────┘   └────┬─────┘   └────┬─────┘
             │              │              │
            EBS            EBS            EBS
             │              │              │
             └──────────────┼──────────────┘
                            │
                    ┌───────┴────────┐
                    ↓                ↓
                  RDS               S3
               PostgreSQL        Files/Objects
```

All of that lives within the AWS networking/security architecture:

```text
                         AWS REGION
                             │
                            VPC
                             │
                ┌────────────┴────────────┐
                │                         │
             AZ-a                      AZ-b
                │                         │
             Subnets                   Subnets
                │                         │
               EC2                       EC2
```

And IAM controls access:

```text
                    IAM
                     │
          ┌──────────┼───────────┐
          ↓          ↓           ↓
       EC2 Role   Developer    Services
          │
          ↓
         S3
```

---

# 20. 🧑‍💻 But where does YOUR code enter this?

This is the part I really want you to understand.

You start here:

```text
                 YOUR CODE
                    │
                    ↓
                 GitHub
                    │
                    ↓
             GitHub Actions
                    │
             ┌──────┴──────┐
             ↓             ↓
           Tests       Docker Build
                           │
                           ↓
                        Docker
                         Image
                           │
                           ↓
                          ECR
                           │
                           ↓
                     EC2 / Deployment
```

And then production:

```text
                         INTERNET
                            │
                            ↓
                        Route 53
                            │
                            ↓
                       CloudFront
                            │
                            ↓
                           ALB
                            │
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
               EC2        EC2        EC2
                 │          │          │
               Docker     Docker     Docker
                 │          │          │
             FastAPI     FastAPI    FastAPI
                 │          │          │
                 └──────┬───┴──────────┘
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
             RDS                 S3
          PostgreSQL          Documents
```

And:

```text
AMI + Launch Template
          ↓
    Auto Scaling Group
          ↓
    creates EC2s
```

---

# 🔥 And THIS is why the concepts you've been learning connect

You didn't learn random AWS services.

You've basically been assembling a production system piece by piece:

```text
Linux
  ↓
Git
  ↓
Docker
  ↓
CI/CD
  ↓
AWS
  │
  ├── IAM       → Security/permissions
  ├── VPC       → Network
  ├── Subnet    → Network segmentation
  ├── EC2       → Compute
  ├── EBS       → Disk
  ├── AMI       → Machine image
  ├── Template  → Launch configuration
  ├── ASG       → Scaling
  ├── ALB       → Traffic distribution
  ├── RDS       → Database
  ├── S3        → Object storage
  ├── Route 53  → DNS
  ├── CloudFront→ CDN
  └── CloudWatch→ Monitoring
```

And the **system-design concepts you already learned** are the architecture sitting above these services:

```text
                    SYSTEM DESIGN
                         │
          ┌──────────────┼───────────────┐
          ↓              ↓               ↓
       Scaling       Availability      Storage
          │              │               │
          ↓              ↓               ↓
         ASG        Multi-AZ/VPC      EBS/RDS/S3
          │
          ↓
      EC2 + AMI
          │
          ↓
     Launch Template
```

So when you eventually design a real production system, you don't start by memorizing AWS services.

You think:

> **"I need compute, persistent storage, a database, networking, security, traffic distribution, scaling, monitoring and deployment."**

Then AWS gives you the building blocks to implement those requirements.

**That's the bridge between System Design → DevOps → Cloud Engineering.**
