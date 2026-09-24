# Amazon RDS — Start Here

## 1. What is RDS?

**Amazon RDS (Relational Database Service)** is a **managed relational database service** provided by AWS.

Instead of creating an EC2 server and manually installing PostgreSQL/MySQL/etc., AWS manages much of the underlying database infrastructure for you.

### Without RDS

You could do:

```text
EC2
 │
 ├── Install Linux
 ├── Install PostgreSQL
 ├── Configure PostgreSQL
 ├── Manage updates
 ├── Configure backups
 ├── Monitor database
 └── Handle failures
```

You are responsible for a lot.

### With RDS

```text
Your Application
       ↓
      RDS
       ↓
PostgreSQL / MySQL / etc.
```

AWS handles much of the infrastructure and operational work around the database.

---

# 2. Why does RDS exist?

Imagine you're building your FastAPI application.

Your application needs to store structured data:

```text
Users
Orders
Products
Payments
Chat history
Documents
```

You could install PostgreSQL yourself on EC2:

```text
FastAPI
   ↓
EC2
   ↓
PostgreSQL
```

But now **you are responsible for the database server**.

You have to think about:

* Database installation
* Patching
* Backups
* Storage
* Monitoring
* Failure recovery
* High availability
* Scaling

RDS removes much of this infrastructure management.

```text
FastAPI
   ↓
RDS
   ↓
PostgreSQL
```

You mainly focus on:

> **Your database, schema, queries, users, and application data.**

AWS handles much of the underlying infrastructure.

---

# 3. RDS is NOT a database engine

This is an important distinction.

**RDS is the managed service.**

Inside RDS, you can choose a database engine such as:

```text
RDS
│
├── PostgreSQL
├── MySQL
├── MariaDB
├── Oracle
└── SQL Server
```

So don't think:

> RDS = PostgreSQL

Instead:

> **RDS = AWS managed platform for relational database engines.**

---

# 4. What does "relational" mean?

A relational database stores structured data in **tables**.

For example:

```text
Users

+----+----------+-------------------+
| id | name     | email             |
+----+----------+-------------------+
| 1  | Imran    | imran@example.com |
| 2  | Ali      | ali@example.com   |
+----+----------+-------------------+
```

Another table:

```text
Orders

+----+---------+--------+
| id | user_id | amount |
+----+---------+--------+
| 1  | 1       | 500    |
| 2  | 2       | 900    |
+----+---------+--------+
```

The tables can be related:

```text
Users
  │
  │ user_id
  ▼
Orders
```

That's where the **relational** part comes from.

---

# 5. RDS vs EC2

This is a very important AWS architectural decision.

### Database on EC2

```text
EC2
│
├── Linux
├── PostgreSQL
├── Database files
└── Your responsibility
```

You manage the database server.

### RDS

```text
AWS
│
└── RDS
     │
     └── PostgreSQL
```

AWS manages much of the underlying database infrastructure.

### Mental model

> **EC2 gives you a server.**

> **RDS gives you a managed relational database.**

---

# 6. What does AWS manage?

With RDS, AWS handles many infrastructure-level tasks such as:

* Database server provisioning
* Underlying infrastructure
* Automated backups
* Software patching options
* Monitoring/integration
* Storage management
* High-availability features
* Failure recovery mechanisms

But **you still manage the database itself**.

For example, you still decide:

```text
What tables should exist?
What columns?
What indexes?
What relationships?
What SQL queries?
Who should access the database?
```

So RDS is **managed**, not completely automatic.

---

# 7. RDS architecture

A simple architecture for your FastAPI backend:

```text
                 Internet
                    │
                    ▼
                   ALB
                    │
                    ▼
              ┌───────────┐
              │   EC2     │
              │ FastAPI   │
              └─────┬─────┘
                    │
                    ▼
              ┌───────────┐
              │    RDS    │
              │ PostgreSQL│
              └───────────┘
```

Notice something important:

### Users don't normally connect directly to RDS.

Instead:

```text
User
 ↓
FastAPI
 ↓
RDS
```

Your backend talks to the database.

---

# 8. RDS and security

This is extremely important.

You generally **don't expose your database directly to the Internet**.

Instead:

```text
Internet
   │
   ▼
ALB
   │
   ▼
EC2
   │
   ▼
RDS
```

Security Groups can enforce this.

For example:

```text
ALB Security Group
        ↓
   allows HTTP/HTTPS
        ↓
EC2 Security Group
        ↓
   allows application traffic
        ↓
RDS Security Group
        ↓
   allows PostgreSQL 5432
   ONLY from EC2 SG
```

So the RDS database doesn't need:

```text
0.0.0.0/0
```

for PostgreSQL.

Instead, you can say conceptually:

> "Only my application servers are allowed to connect to this database."

That's a much better architecture.

---

# 9. RDS and EBS

Here's another connection to what you've already learned.

RDS still needs underlying storage.

But AWS manages that storage infrastructure for you.

Conceptually:

```text
RDS
 │
 └── Database
       │
       └── Managed storage
```

You don't normally SSH into an RDS instance and manipulate its underlying disk like you would with your EC2 + EBS setup.

That's one of the major differences between **self-managed database on EC2** and **RDS**.

---

# 10. RDS backups

RDS supports **automated backups**.

You can configure a backup retention period.

For example:

```text
Monday
   ↓
Tuesday
   ↓
Wednesday
   ↓
Thursday
   ↓
...
```

RDS can maintain backups according to your configured retention settings.

You can also create **manual DB snapshots**.

This gives you two important concepts:

```text
Automated backups
→ Managed according to retention settings

DB snapshots
→ Manually created database backup
```

We'll go deeper into both later.

---

# 11. High Availability

One of the biggest reasons to use RDS is that AWS provides features for **high availability**.

A major feature is:

### Multi-AZ

Conceptually:

```text
Availability Zone A
┌─────────────────┐
│ RDS Primary     │
└─────────────────┘
        │
        │ replication
        ▼
Availability Zone B
┌─────────────────┐
│ Standby         │
└─────────────────┘
```

If the primary database has a failure, AWS can perform a failover to the standby.

The important concept:

> **Multi-AZ is primarily about availability/failover, not simply making queries faster.**

We'll study this properly later.

---

# 12. Read Replicas

This is a different concept.

Suppose your application receives:

```text
10,000 READ queries
1,000 WRITE queries
```

You can use read replicas to handle additional read workload.

Conceptually:

```text
             ┌──→ Primary RDS
             │
Application ─┤
             │
             ├──→ Read Replica 1
             │
             └──→ Read Replica 2
```

### Multi-AZ vs Read Replica

Remember:

```text
Multi-AZ
→ High availability / failover

Read Replica
→ Read scaling
```

This distinction is **very important for AWS architecture questions**.

---

# 13. RDS vs S3

You just learned S3, so compare them.

| RDS                         | S3                              |
| --------------------------- | ------------------------------- |
| Relational database         | Object storage                  |
| Tables/rows/columns         | Objects                         |
| SQL queries                 | Object/API operations           |
| Structured application data | Files/data objects              |
| PostgreSQL/MySQL/etc.       | PDFs/images/videos/backups/etc. |

For your AI application:

```text
FastAPI
 │
 ├──→ RDS
 │     └── Users, metadata, conversations
 │
 └──→ S3
       └── PDFs, images, uploaded files
```

---

# 🧠 The core RDS mental model

Remember this:

> **RDS = AWS manages the infrastructure around a relational database so you don't have to operate the database server yourself.**

And your architecture becomes:

```text
             Internet
                │
                ▼
               ALB
                │
                ▼
          FastAPI / EC2
           │          │
           │          │
           ▼          ▼
          RDS        S3
       structured   files/objects
         data
```

### Our RDS learning sequence

I'd suggest we learn RDS in this order:

```text
1. RDS fundamentals          ← we're here
2. Database engines
3. RDS instance architecture
4. DB subnet groups
5. Security Groups
6. Public vs private RDS
7. Storage
8. Automated backups
9. DB snapshots
10. Multi-AZ
11. Read Replicas
12. RDS scaling
13. RDS Proxy
14. Encryption
15. Monitoring
16. Hands-on: create PostgreSQL RDS
17. Connect FastAPI → RDS
```

The **next concept I'd learn is RDS architecture: DB instance, endpoint, port, VPC, subnet group, and how your FastAPI server actually connects to RDS.**
