# DynamoDB — Complete Notes

## 1. What is DynamoDB?

Amazon Web Services DynamoDB is a **fully managed NoSQL database service provided by AWS**.

Unlike your RDS MySQL database:

```text
RDS MySQL
    ↓
Relational database
    ↓
Tables → Rows → Columns
```

DynamoDB is:

```text
DynamoDB
    ↓
NoSQL
    ↓
Tables → Items → Attributes
```

DynamoDB supports both:

* **Key-value data**
* **Document-style data**

And AWS manages the underlying infrastructure, scaling, availability, patching, etc.

---

# 2. First: What is NoSQL?

**NoSQL** is commonly interpreted as **"Not Only SQL."**

It isn't simply:

> "A database that doesn't use SQL."

It's a broad family of database models designed for workloads where the relational model isn't always the best fit.

Common NoSQL models:

```text
NoSQL
 │
 ├── Key-Value
 │
 ├── Document
 │
 ├── Wide-Column
 │
 └── Graph
```

DynamoDB primarily provides:

```text
Key-Value + Document
```

---

# 3. SQL vs NoSQL

Your MySQL database looked like:

```text
users

id | username | email
----------------------
1  | Imran    | ...
2  | Ali      | ...
3  | Ahmed    | ...
```

Relational databases emphasize:

* Tables
* Relationships
* Joins
* Structured schemas
* SQL
* Transactions
* Constraints

NoSQL databases often emphasize:

* Flexible data models
* High-scale distributed workloads
* Access patterns
* Horizontal scaling
* Low/predictable latency for suitable workloads

Neither is universally "better."

A typical system can use both.

For example:

```text
FastAPI
   │
   ├── RDS PostgreSQL → users, orders, billing
   │
   ├── DynamoDB → high-scale application state
   │
   └── S3 → files/images/PDFs
```

---

# 4. Why does DynamoDB still have "Tables"?

You asked this earlier.

Yes, DynamoDB has **tables**.

But the meaning is different from a traditional relational table.

### SQL

```text
Table
 │
 ├── Row
 ├── Row
 └── Row
```

### DynamoDB

```text
Table
 │
 ├── Item
 ├── Item
 └── Item
```

And each item contains **attributes**.

So:

```text
SQL:
Row     → Column

DynamoDB:
Item    → Attribute
```

---

# 5. Item

An **item** is roughly comparable to a row in SQL.

For example:

```json
{
  "user_id": "123",
  "username": "Imran",
  "age": 22
}
```

That's one DynamoDB **item**.

Another item could be:

```json
{
  "user_id": "456",
  "username": "Ali"
}
```

Notice that the second item doesn't have to contain `age`.

That's part of DynamoDB's flexible/schemaless model.

---

# 6. Attribute

An **attribute** is a piece of data inside an item.

For example:

```json
{
  "user_id": "123",
  "username": "Imran",
  "age": 22
}
```

Attributes are:

```text
user_id
username
age
```

Unlike a traditional SQL table, DynamoDB doesn't require every item to have exactly the same set of non-key attributes.

---

# 7. Schemaless

DynamoDB describes itself as **schemaless**.

That means you don't define a rigid set of columns for every item like:

```text
users
-------------------
id       INT
username VARCHAR
email    VARCHAR
age      INT
```

Instead, items can have different attributes.

For example:

```json
{
  "user_id": "1",
  "username": "Imran"
}
```

and:

```json
{
  "user_id": "2",
  "username": "Ali",
  "country": "Pakistan",
  "skills": ["Python", "Docker"]
}
```

can exist in the same table.

### But!

Schemaless does **not** mean:

> "There is no structure."

The table still has a required **primary key structure**.

---

# 8. Primary Key

Every DynamoDB table needs a **primary key**.

There are two possibilities:

```text
Simple primary key
       ↓
Partition key

OR

Composite primary key
       ↓
Partition key + Sort key
```

This is one of the most important DynamoDB concepts.

---

# 9. Partition Key ⭐

Suppose we create:

```text
Table: Users

Partition key:
user_id
```

Items:

```json
{
  "user_id": "1",
  "username": "Imran"
}
```

```json
{
  "user_id": "2",
  "username": "Ali"
}
```

```json
{
  "user_id": "3",
  "username": "Ahmed"
}
```

The `user_id` is the **partition key**.

---

# 10. Why "partition"?

Because DynamoDB is designed to operate at large scale.

Conceptually:

```text
                 DynamoDB
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
      Partition  Partition  Partition
```

DynamoDB uses the partition-key value to determine how data is distributed across its underlying infrastructure.

You don't manually select the physical machine.

AWS handles that.

Think of the partition key as an **addressing mechanism** that helps DynamoDB locate and distribute data.

---

# 11. Partition Key Uniqueness

If your table uses only:

```text
user_id
```

as its primary key, each item must have a unique `user_id`.

You can have:

```text
1
2
3
```

but not two items whose complete primary key is:

```text
1
```

because that would be the same item key.

---

# 12. Sort Key ⭐

The sort key is **optional**.

You can create a table with:

```text
Partition key = user_id
```

or:

```text
Partition key = user_id
Sort key = order_id
```

The second design is called a **composite primary key**.

---

# 13. Why would we need a Sort Key?

Imagine an e-commerce application.

You want multiple orders for each user.

```text
user_id | order_id
-------------------
1       | 101
1       | 102
1       | 103

2       | 201
2       | 202
```

Here:

```text
Partition Key = user_id
Sort Key      = order_id
```

Now `user_id = 1` can appear multiple times because the **combination** must be unique:

```text
(1,101)
(1,102)
(1,103)
```

---

# 14. Why is it called Sort Key?

Items sharing the same partition key can be organized/queryable according to their sort-key values.

For example:

```text
user_id | created_at
---------------------------
1       | 2026-09-01
1       | 2026-09-10
1       | 2026-09-20
```

You could query:

```text
user_id = 1
AND created_at > 2026-09-10
```

This is why the sort key becomes extremely useful for things like:

* Orders
* Messages
* Events
* Transactions
* Time-series data

---

# 15. The BIG DynamoDB design idea: Access Patterns

This is one of the biggest differences from traditional relational database design.

With SQL, you might start thinking:

```text
What entities do I have?

User
Order
Product
Payment
```

Then design normalized tables and relationships.

With DynamoDB, you should heavily consider:

> **How will my application access the data?**

For example:

```text
Requirement:

"Get all orders for user 123."
```

That requirement influences your key design.

You might choose:

```text
Partition key = user_id
Sort key = order_id
```

So DynamoDB design is heavily **access-pattern driven**.

---

# 16. Capacity

Now we get to the console screen you showed me.

DynamoDB has:

```text
Capacity
 │
 ├── Reads
 │
 └── Writes
```

Because applications perform:

```text
READ
WRITE
```

operations.

DynamoDB measures capacity using:

```text
RCU = Read Capacity Unit

WCU = Write Capacity Unit
```

Don't think:

> 1 RCU = exactly one read.

Capacity consumption depends on things such as **item size** and the consistency model.

---

# 17. Capacity Modes

DynamoDB provides two major capacity modes:

```text
Capacity Mode
    │
    ├── On-demand
    │
    └── Provisioned
```

---

# 18. On-demand

Your console currently showed:

> **On-demand — pay for the actual reads and writes your application performs.**

Mental model:

```text
Traffic unpredictable
       ↓
On-demand
       ↓
AWS handles capacity automatically
       ↓
Pay based on usage
```

For a new application where you don't know the workload yet, this is often the simpler model.

---

# 19. Provisioned

With provisioned capacity, you configure expected read/write capacity.

Mental model:

```text
You know your workload
        ↓
Estimate required capacity
        ↓
Provision capacity
        ↓
Application uses it
```

For example, conceptually:

```text
Expected workload:

100 reads/second
20 writes/second
```

You configure capacity accordingly.

---

# 20. On-demand vs Provisioned

|               | On-demand                 | Provisioned                |
| ------------- | ------------------------- | -------------------------- |
| Capacity      | AWS handles automatically | You configure it           |
| Workload      | Variable/unpredictable    | Predictable                |
| Management    | Simpler                   | More control               |
| Billing model | Usage-based               | Provisioned capacity model |

For your learning table:

**On-demand is perfectly fine.**

---

# 21. Maximum Table Throughput

You also saw:

```text
Maximum read request units
Maximum write request units
```

This is an optional ceiling for on-demand throughput.

Conceptually:

```text
On-demand
    ↓
AWS automatically handles capacity
    ↓
But you can optionally define
a maximum throughput limit
```

This can help bound usage/cost.

---

# 22. Global Tables 🌍

Now we're moving into advanced DynamoDB.

Normally:

```text
DynamoDB
   ↓
us-east-1
   ↓
Table
```

But suppose your application operates globally.

You can use **Global Tables**.

Conceptually:

```text
                  Global Table
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
      us-east-1    eu-west-1   ap-south-1
```

The DynamoDB table is replicated across multiple AWS Regions.

This is useful for:

* Global applications
* Regional resilience
* Lower latency for geographically distributed users
* Multi-Region architectures

### Mental model:

> **Global Tables = DynamoDB data across multiple Regions.**

---

# 23. DAX ⚡

DAX stands for:

**DynamoDB Accelerator**

DAX is a **caching service designed for DynamoDB**.

Without DAX:

```text
Application
     ↓
DynamoDB
```

With DAX:

```text
Application
     ↓
    DAX
     ↓
DynamoDB
```

Suppose your application repeatedly asks:

```text
"Give me product 123."
```

DAX can cache frequently accessed data.

First request:

```text
Application
    ↓
DAX
    ↓
DynamoDB
    ↓
DAX caches result
```

Later request:

```text
Application
    ↓
DAX
    ↓
cached result
```

So:

> **DAX = caching/low-latency acceleration for DynamoDB workloads.**

---

# 24. Global Tables vs DAX

Don't mix these two up.

```text
GLOBAL TABLES
      ↓
🌍 Multiple Regions
      ↓
Replication
```

while:

```text
DAX
      ↓
⚡ Cache
      ↓
Faster repeated reads
```

| Feature      | Global Tables                    | DAX                         |
| ------------ | -------------------------------- | --------------------------- |
| Main purpose | Multi-Region                     | Caching                     |
| Geography    | Multiple Regions                 | Not the main purpose        |
| Main benefit | Regional distribution/resilience | Lower-latency cached access |
| Replication  | Yes                              | No, that's not its purpose  |
| Cache        | No                               | Yes                         |

---

# 25. Other settings you saw

When creating your DynamoDB table, AWS showed several additional settings.

### Table class

You saw:

```text
DynamoDB Standard
```

This relates to the storage/pricing characteristics of the table.

For now, Standard is fine.

---

### Local Secondary Index — LSI

An **LSI** gives another way to query items while using the same partition key but a different sort key.

You don't need to master this yet.

Important thing you saw:

> **LSIs must be defined when the table is created.**

---

### Global Secondary Index — GSI

A **GSI** gives another key structure for querying the same underlying table.

Suppose your main table is designed around:

```text
user_id
```

but your application also frequently needs:

```text
Find user by email
```

A GSI can provide another access pattern.

GSI is an important DynamoDB concept that you'll want to learn properly.

---

### Encryption

Your console showed:

```text
AWS-owned key
```

DynamoDB supports encryption at rest.

AWS manages the encryption infrastructure for you.

You can later learn AWS KMS and customer-managed keys.

---

### Deletion protection

You saw:

```text
Deletion protection: Off
```

When enabled, it helps protect a table from accidental deletion.

---

### Tags

Tags are simple:

```text
Key: Environment
Value: Dev
```

or:

```text
Key: Project
Value: Dynamo-Learning
```

They can help with organization, cost tracking, and access-control scenarios.

---

# 26. Your DynamoDB mental model

If you forget everything else, remember this:

```text
                     DynamoDB
                        │
                        ↓
                      Table
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
           Item                 Item
              │                   │
        ┌─────┴─────┐       ┌─────┴─────┐
        ↓           ↓       ↓           ↓
    Attribute   Attribute  Attribute  Attribute
```

Every table has a primary key:

```text
Primary Key
     │
     ├── Partition Key
     │
     └── Sort Key (optional)
```

---

# 27. The complete picture

Now put everything together:

```text
                         DynamoDB
                            │
                     ┌──────┴──────┐
                     ↓             ↓
                   Table        Capacity
                     │             │
                     ↓        ┌────┴────┐
                   Items      ↓         ↓
                     │      Reads     Writes
                     ↓        │         │
                Attributes    RCU       WCU
                     │
                     ↓
                Primary Key
                     │
              ┌──────┴──────┐
              ↓             ↓
        Partition Key    Sort Key
              │          (optional)
              │
              ↓
        Data distribution
        + efficient access
```

Then advanced features:

```text
DynamoDB
   │
   ├── Global Tables 🌍
   │       └── Multi-Region replication
   │
   └── DAX ⚡
           └── Caching
```

---

# 28. DynamoDB vs your RDS MySQL

This is probably the most useful comparison for you because you've actually built both.

### Your RDS MySQL app

```text
FastAPI
   ↓
RDS MySQL
   ↓
simple_app
   ↓
users table
   ↓
rows + columns
```

You think:

```text
tables
relationships
SQL
joins
schema
transactions
```

### DynamoDB

```text
Application
    ↓
DynamoDB
    ↓
Users table
    ↓
Items
    ↓
Attributes
```

You think:

```text
access patterns
partition key
sort key
Query
indexes
distributed scale
capacity
```

---

# 29. The most important concepts to remember

If I had to reduce everything we've learned to **10 things**, I'd give you these:

1. **DynamoDB is AWS's managed NoSQL database.**
2. **DynamoDB tables contain items, not traditional SQL rows.**
3. **Items contain attributes.**
4. **DynamoDB is schemaless/flexible for non-key attributes.**
5. **Every table needs a primary key.**
6. **The primary key can be a partition key alone, or partition key + sort key.**
7. **Partition keys are fundamental to data distribution and efficient access.**
8. **DynamoDB capacity is measured through reads/writes, including RCU/WCU concepts.**
9. **On-demand vs provisioned controls how capacity is managed/billed.**
10. **Global Tables = multi-Region; DAX = caching.**

And there's one sentence I especially want you to remember:

> **With DynamoDB, don't just ask "What data do I have?" Ask "How will my application access this data?"**

That's the mindset that separates **using DynamoDB** from actually **designing with DynamoDB**.
