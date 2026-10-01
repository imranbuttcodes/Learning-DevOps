This is the point where we should build the **complete AWS networking mental model**, not memorize isolated definitions.

The most important thing first:

# 1. The overall hierarchy

Think of AWS networking like this:

```text
AWS
│
└── Region
    │
    └── VPC
        │
        ├── Availability Zone
        │   │
        │   ├── Public Subnet
        │   │   └── Resources
        │   │
        │   └── Private Subnet
        │       └── Resources
        │
        ├── Route Tables
        │
        ├── Internet Gateway
        │
        ├── NAT Gateway
        │
        ├── Network ACLs
        │
        ├── Security Groups
        │
        ├── VPC Endpoints
        │
        ├── VPC Peering
        │
        ├── VPC Flow Logs
        │
        ├── Bastion Host
        │
        ├── Elastic IP
        │
        ├── AWS Client VPN
        │
        └── Direct Connect
```

But don't interpret every item as literally being "inside" the VPC. Some are associated with it, some provide connectivity to it, and some are separate AWS networking services.

---

# 2. Region

A **Region** is a geographic AWS location containing multiple Availability Zones.

For example:

```text
AWS
│
├── us-east-1
├── us-west-2
├── eu-west-1
├── ap-southeast-1
└── ...
```

Suppose we're using:

```text
us-east-1
```

Inside it:

```text
us-east-1
│
├── AZ
├── AZ
├── AZ
├── ...
```

### Why Regions?

Primarily:

* Geographic separation
* Data residency
* Latency
* Disaster recovery
* Regulatory requirements

For example:

```text
Users in Pakistan
       │
       ▼
AWS Region closer to users
```

---

# 3. Availability Zone

An **Availability Zone (AZ)** is an isolated infrastructure location within a Region.

Conceptually:

```text
Region
│
├── AZ-1
│
├── AZ-2
│
└── AZ-3
```

Each AZ consists of one or more physically separate data centers.

### Why?

High availability.

Instead of:

```text
Application
    │
    ▼
  EC2
   AZ-1
```

you can have:

```text
             Load Balancer
              /         \
             ▼           ▼
          EC2-A        EC2-B
           AZ-1         AZ-2
```

If AZ-1 has an infrastructure failure, AZ-2 can continue serving traffic.

---

# 4. VPC

Now we enter the really important part.

**VPC = Virtual Private Cloud.**

A VPC is your logically isolated network inside an AWS Region.

Think of it as:

> **"My own virtual network in AWS."**

Example:

```text
Region: us-east-1

┌───────────────────────────────────┐
│              My VPC               │
│                                   │
│     10.0.0.0/16                   │
│                                   │
└───────────────────────────────────┘
```

The `10.0.0.0/16` is the VPC's **CIDR block** — its IP address range.

Inside the VPC you create:

```text
VPC
│
├── Subnets
├── Route Tables
├── Security Groups
├── Network ACLs
└── connectivity components
```

---

# 5. Subnet

A **subnet** is a smaller IP range inside your VPC.

Suppose:

```text
VPC
10.0.0.0/16
```

You could divide it:

```text
VPC: 10.0.0.0/16
│
├── Public Subnet
│   10.0.1.0/24
│
├── Public Subnet
│   10.0.2.0/24
│
├── Private Subnet
│   10.0.11.0/24
│
└── Private Subnet
    10.0.12.0/24
```

### Very important:

A subnet belongs to **one AZ**.

So:

```text
Region
│
├── AZ-1
│   ├── Public Subnet
│   └── Private Subnet
│
└── AZ-2
    ├── Public Subnet
    └── Private Subnet
```

---

# 6. Public vs Private Subnet

This is where many beginners get confused.

A subnet is considered **public** when its route table has a route to an Internet Gateway.

Example:

```text
Public Subnet
      │
      ▼
Route Table
      │
      │ 0.0.0.0/0 → IGW
      ▼
Internet Gateway
      │
      ▼
Internet
```

Private:

```text
Private Subnet
      │
      ▼
Route Table
      │
      │ 0.0.0.0/0 → NAT
      ▼
NAT Gateway
      │
      ▼
Internet Gateway
      │
      ▼
Internet
```

A **private subnet does not have a direct route to the Internet Gateway**.

---

# 7. Route Table

Now we need to answer:

> **How does AWS know where network traffic should go?**

That's the job of a **route table**.

A route table contains rules like:

```text
Destination       Target
────────────────────────────
10.0.0.0/16       local
0.0.0.0/0         igw-xxxx
```

Meaning:

```text
10.0.0.0/16
     ↓
Stay inside VPC

0.0.0.0/0
     ↓
Send elsewhere through IGW
```

### Example

Public subnet:

```text
Route Table
│
├── 10.0.0.0/16 → local
└── 0.0.0.0/0   → Internet Gateway
```

Private subnet:

```text
Route Table
│
├── 10.0.0.0/16 → local
└── 0.0.0.0/0   → NAT Gateway
```

This is one of the most important networking concepts.

---

# 8. Internet Gateway — IGW

An **Internet Gateway** connects a VPC to the Internet.

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
VPC
```

But simply attaching an IGW doesn't automatically make everything public.

You need the correct:

1. Route table
2. Public IP addressing
3. Security rules

For example:

```text
Internet
   │
   ▼
 IGW
   │
   ▼
Public Subnet
   │
   ▼
 EC2
```

---

# 9. Security Group

You missed this in your original list, but **this is one of the most important ones.**

A Security Group is a **stateful virtual firewall associated with resources such as EC2 network interfaces**.

Example:

```text
Internet
   │
   ▼
Security Group
   │
   ▼
EC2
```

Rules:

```text
Inbound:
TCP 80   → 0.0.0.0/0
TCP 443  → 0.0.0.0/0
TCP 22   → your IP
```

So:

```text
Internet
   │
   ├── HTTP 80  → allowed
   ├── HTTPS 443 → allowed
   └── SSH 22   → only your IP
```

Security Groups are **stateful**.

---

# 10. Network ACL — NACL

NACL = **Network Access Control List**.

It is another firewall-like mechanism, but its scope is different.

```text
VPC
│
└── Subnet
      │
      ▼
     NACL
      │
      ▼
   Resources
```

### Security Group

```text
Resource-level
```

### NACL

```text
Subnet-level
```

NACLs are **stateless**.

That means inbound and outbound traffic are evaluated separately.

Example:

```text
Internet
   │
   ▼
NACL
   │
   ▼
Subnet
   │
   ▼
EC2
```

---

# 11. NAT Gateway

NAT = **Network Address Translation**.

The common AWS use case:

> Allow resources in a private subnet to access the Internet **outbound**, without allowing the Internet to initiate connections to those resources.

Architecture:

```text
              INTERNET
                  ▲
                  │
             Internet
              Gateway
                  ▲
                  │
             NAT Gateway
                  ▲
                  │
           Private Subnet
                  │
                EC2
```

Example:

Your private EC2 needs to:

```text
sudo dnf update
```

It needs Internet access to download packages.

Instead of making EC2 public:

```text
Private EC2
     ↓
NAT Gateway
     ↓
Internet
```

The EC2 remains private.

---

# 12. VPC Endpoint

Now suppose private EC2 needs to access an AWS service such as S3.

Instead of:

```text
Private EC2
     ↓
NAT
     ↓
Internet
     ↓
S3
```

you can use a VPC Endpoint.

```text
Private EC2
     │
     ▼
VPC Endpoint
     │
     ▼
S3
```

This provides private connectivity to supported AWS services.

There are different endpoint types, including **gateway endpoints** and **interface endpoints**.

You don't need to memorize all of that yet.

Mental model:

> **VPC Endpoint = private path from your VPC to supported AWS services.**

---

# 13. VPC Peering

Suppose you have two VPCs:

```text
VPC-A
10.0.0.0/16

VPC-B
10.1.0.0/16
```

Normally they're isolated.

VPC Peering creates a private connection:

```text
┌──────────────┐
│    VPC-A     │
│ 10.0.0.0/16  │
└──────┬───────┘
       │
       │ Peering
       │
┌──────▼───────┐
│    VPC-B     │
│ 10.1.0.0/16  │
└──────────────┘
```

Then you need appropriate routes in the route tables.

---

# 14. Bastion Host

A Bastion Host is basically a controlled entry point into private infrastructure.

Example:

```text
Internet
   │
   ▼
Bastion
Public Subnet
   │
   ▼
Private EC2
Private Subnet
```

You SSH into the bastion:

```text
ssh ec2-user@bastion
```

Then from the bastion:

```text
ssh ec2-user@private-ec2
```

The private EC2 doesn't need a public IP.

### Modern AWS alternative

AWS Systems Manager Session Manager can often eliminate the need for a traditional bastion host.

So understand the **concept**, but don't treat Bastion as mandatory.

---

# 15. Elastic IP

An **Elastic IP (EIP)** is a static public IPv4 address that you can associate with supported AWS resources.

Without one, an EC2 public IPv4 address can change after certain lifecycle operations.

Conceptually:

```text
Elastic IP
    │
    ▼
   EC2
```

Example:

```text
44.207.14.79
```

You actually experimented with this during your SSH troubleshooting.

### Important

Elastic IPs are public addresses and AWS may charge for public IPv4 usage under current pricing rules, so don't allocate them unnecessarily.

---

# 16. VPC Flow Logs

Now we move from **controlling traffic** to **observing traffic**.

VPC Flow Logs capture information about network traffic for supported resources.

Conceptually:

```text
EC2
 │
 │ network traffic
 ▼
VPC Flow Logs
 │
 ├── CloudWatch Logs
 └── S3
```

You can use them for:

* Troubleshooting
* Security investigation
* Network analysis

Example:

```text
"Why can't EC2-A connect to EC2-B?"
```

Flow Logs can help determine whether traffic was accepted/rejected at the network interface level.

---

# 17. AWS Client VPN

Client VPN is for connecting **individual users/devices** to AWS resources through a VPN.

Example:

```text
Developer laptop
       │
       │ VPN
       ▼
AWS Client VPN
       │
       ▼
VPC
       │
       ▼
Private EC2
```

So a developer sitting at home can securely access private AWS resources without making those resources public.

---

# 18. AWS Direct Connect

This is more enterprise-oriented.

Suppose a company has its own physical data center:

```text
Company Data Center
        │
        │ Dedicated network connection
        │
        ▼
AWS Direct Connect
        │
        ▼
AWS
        │
        ▼
VPC
```

It provides a dedicated network connection between an on-premises network and AWS.

It's different from a normal VPN connection over the public Internet.

---

# 19. Put everything together

Now let's build the architecture.

```text
                         INTERNET
                            │
                            ▼
                  ┌──────────────────┐
                  │ Internet Gateway │
                  └────────┬─────────┘
                           │
                ┌──────────▼──────────┐
                │        VPC          │
                │    10.0.0.0/16      │
                │                     │
                │  ┌───────────────┐  │
                │  │ Public Subnet │  │
                │  │   AZ-1        │  │
                │  │               │  │
                │  │     ALB       │  │
                │  │      │        │  │
                │  └──────┼────────┘  │
                │         │           │
                │  ┌──────▼────────┐  │
                │  │Private Subnet │  │
                │  │   AZ-1        │  │
                │  │               │  │
                │  │    EC2        │  │
                │  │   FastAPI     │  │
                │  └──────┬────────┘  │
                │         │           │
                │  ┌──────▼────────┐  │
                │  │Private Subnet │  │
                │  │   AZ-2        │  │
                │  │               │  │
                │  │     RDS       │  │
                │  └───────────────┘  │
                │                     │
                └─────────────────────┘
```

Then add the supporting networking pieces:

```text
VPC
│
├── Subnets
│
├── Route Tables
│
├── Internet Gateway
│
├── NAT Gateway
│
├── Security Groups
│
├── Network ACLs
│
├── VPC Endpoints
│
├── VPC Peering
│
└── Flow Logs
```

And outside/around the VPC:

```text
Bastion Host
Elastic IP
Client VPN
Direct Connect
```

---

# 20. The most important hierarchy to memorize

Don't memorize the entire giant diagram.

Memorize this:

```text
AWS Region
   │
   └── VPC
        │
        ├── AZ
        │    │
        │    └── Subnet
        │         │
        │         └── Resources
        │
        ├── Route Table
        │
        ├── Security Group
        │
        ├── NACL
        │
        ├── Internet Gateway
        │
        ├── NAT Gateway
        │
        ├── VPC Endpoint
        │
        └── VPC Peering
```

And remember the **jobs**:

| Component            | Main job                             |
| -------------------- | ------------------------------------ |
| **Region**           | Geographic AWS location              |
| **AZ**               | Isolated infrastructure location     |
| **VPC**              | Your virtual network                 |
| **Subnet**           | IP range inside VPC, tied to one AZ  |
| **Route Table**      | Decides where traffic goes           |
| **Internet Gateway** | VPC ↔ Internet                       |
| **Security Group**   | Stateful resource-level firewall     |
| **NACL**             | Stateless subnet-level firewall      |
| **NAT Gateway**      | Private subnet → Internet outbound   |
| **VPC Endpoint**     | Private VPC → supported AWS services |
| **VPC Peering**      | VPC ↔ VPC private connection         |
| **Bastion**          | Entry point to private servers       |
| **Elastic IP**       | Static public IPv4                   |
| **Flow Logs**        | Network traffic visibility           |
| **Client VPN**       | Users → private VPC                  |
| **Direct Connect**   | On-premises network → AWS            |

### The big picture

And this is why your earlier **FastAPI + RDS** question connects directly to VPC:

```text
Internet
   │
   ▼
Public entry point
(ALB / CloudFront)
   │
   ▼
Private EC2
(FastAPI)
   │
   ▼
Private RDS
(Database)
```

The **Internet-facing API does not mean your database needs to be Internet-facing**.

That's one of the biggest architectural ideas in AWS networking.
