# Amazon S3 — Complete Revision Notes

## 1. What is Amazon S3?

**Amazon S3 (Simple Storage Service)** is AWS's **object storage service**.

It is used to store files/data such as:

* Images
* Videos
* PDFs
* Documents
* Backups
* Logs
* Dataset files
* User uploads
* Static website files

### Mental model

> **S3 = a highly scalable object-storage system where data is stored as objects inside buckets.**

```text
AWS Account
    │
    └── S3
         │
         ├── Bucket A
         │    ├── image.jpg
         │    ├── report.pdf
         │    └── data.json
         │
         └── Bucket B
              ├── video.mp4
              └── backup.zip
```

---

# 2. Bucket

A **bucket** is a container for objects.

Example:

```text
Bucket:
backup-bucket-imran
```

Inside:

```text
backup-bucket-imran/
├── backup.zip
├── report.pdf
└── images/
    ├── img1.jpg
    └── img2.jpg
```

Important:

* Bucket names must be **globally unique**.
* A bucket is created in a specific AWS Region.
* Objects are stored inside buckets.

---

# 3. Object

An **object** is the actual piece of data stored in S3.

For example:

```text
report.pdf
```

An S3 object consists conceptually of:

```text
Object
├── Data
├── Key
├── Metadata
└── Other properties
```

### Object Key

The **key** identifies the object within the bucket.

Example:

```text
images/profile.jpg
```

S3 doesn't actually have traditional folders like a Linux filesystem. The `/` in:

```text
images/profile.jpg
```

is part of the object's **key**. The console presents prefixes such as `images/` in a folder-like way.

---

# 4. S3 vs EBS

This is extremely important.

| EBS                           | S3                                     |
| ----------------------------- | -------------------------------------- |
| Block storage                 | Object storage                         |
| Usually attached to EC2       | Accessed through S3 APIs/HTTP/SDKs     |
| Behaves like a disk           | Doesn't behave like a normal disk      |
| `cd`, `ls`, filesystem        | Bucket + objects                       |
| OS/application/database files | PDFs, images, videos, backups, uploads |
| AZ-specific                   | Regional service                       |

### Mental model

```text
EC2
 │
 └── EBS
      ↓
   "My server's disk"

S3
 ↓
"My application's object storage"
```

---

# 5. Accessing S3 Objects

An object can be accessed through AWS APIs, SDKs, CLI, or URLs depending on its permissions.

For example:

```text
https://bucket-name.s3.us-east-1.amazonaws.com/image.jpg
```

But simply knowing the URL **doesn't mean everyone can access the object**.

Permissions determine whether access is allowed.

---

# 6. Pre-Signed URL

A **pre-signed URL** is a temporary URL that gives someone permission to access a private S3 object.

Example:

```text
FastAPI
   │
   │ Generate pre-signed URL
   ▼
S3
   │
   ▼
Temporary URL
   │
   ▼
User
```

Example use case:

A user wants to download:

```text
private-report.pdf
```

Instead of making the bucket public, your backend generates a URL that works for a limited time.

For example:

```text
Expires = 300 seconds
```

After that, the URL stops working.

### Mental model

> **Pre-signed URL = temporary permission embedded in a URL.**

---

# 7. Bucket Policy

A **bucket policy** is a JSON-based resource policy that controls access to an S3 bucket.

Example structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

Important concepts:

### Effect

```text
Allow
Deny
```

### Principal

**Who** is affected?

Examples:

```text
*
```

means everyone.

Or a specific AWS identity/service.

### Action

**What can they do?**

Examples:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

### Resource

**Which S3 resource?**

---

# 8. Block Public Access

S3 has **Block Public Access** settings designed to prevent accidental public exposure.

You can have a bucket policy saying:

```text
Allow public access
```

but Block Public Access can prevent that public access from taking effect.

### Mental model

```text
Bucket Policy
      +
Block Public Access
      ↓
Final access behavior
```

For production, you generally don't disable public access protection casually.

---

# 9. Static Website Hosting

S3 can host a **static website**.

Static files include:

```text
HTML
CSS
JavaScript
Images
```

Example:

```text
index.html
style.css
script.js
logo.png
```

S3 can serve these files without an EC2 server.

### Static website

```text
User
 ↓
S3
 ↓
index.html
```

But S3 static website hosting doesn't execute backend code like:

```text
FastAPI
Django
Node.js
PHP
```

For a production architecture, you might use:

```text
CloudFront
    ↓
S3
```

for CDN delivery and HTTPS.

---

# 10. S3 Storage Classes

Storage classes determine **how S3 stores your data based on access patterns and cost considerations**.

### S3 Standard

For frequently accessed data.

```text
Frequently accessed
        ↓
S3 Standard
```

Example:

```text
Website assets
Active application files
Frequently accessed images
```

---

### S3 Standard-IA

**IA = Infrequent Access**

For data that isn't accessed often but still needs relatively quick access when requested.

```text
Less frequently accessed
        ↓
Standard-IA
```

---

### S3 One Zone-IA

Similar idea to Standard-IA but stored in **one Availability Zone**.

```text
Infrequently accessed
        +
Can tolerate single-AZ storage
        ↓
One Zone-IA
```

---

### Glacier classes

Designed for archival data.

```text
Glacier Instant Retrieval
Glacier Flexible Retrieval
Glacier Deep Archive
```

The general idea:

```text
More archival
     ↓
Lower storage cost
     ↓
Potentially greater retrieval delay / different retrieval economics
```

---

# 11. Lifecycle Rules

Lifecycle rules automatically manage objects as they become older.

Example:

```text
Day 0
  ↓
S3 Standard
  ↓
Day 30
  ↓
Standard-IA
  ↓
Day 60
  ↓
One Zone-IA
  ↓
Day 365
  ↓
Delete
```

The rule basically says:

> **"When an object reaches a certain age, automatically perform an action."**

Actions can include:

* Transition to another storage class
* Expire/delete current objects
* Manage noncurrent versions
* Delete expired delete markers
* Abort incomplete multipart uploads

### Important

The transition timing is based on **object creation time** for the current-version transitions you configured.

So:

```text
Day 0 → Standard
Day 30 → Standard-IA
Day 60 → One Zone-IA
```

The object isn't duplicated into three classes.

It transitions:

```text
Standard
   ↓
Standard-IA
   ↓
One Zone-IA
```

---

# 12. S3 Versioning

**Versioning** allows S3 to keep multiple versions of the same object.

Without versioning:

```text
report.pdf
   ↓
replace
   ↓
new report.pdf
```

The old version can be lost.

With versioning:

```text
report.pdf
│
├── Version 1
├── Version 2
└── Version 3 ← current
```

This helps protect against:

* Accidental overwrites
* Accidental deletion
* Application mistakes

---

# 13. Delete Marker

When **versioning is enabled**, deleting an object doesn't necessarily immediately destroy the previous versions.

S3 can create a **delete marker**.

Conceptually:

```text
Version 1
Version 2
Version 3
    ↓
Delete object
    ↓
Delete Marker
```

The object appears deleted normally, but older versions can still exist.

---

# 14. S3 Replication

Replication automatically copies objects from one S3 bucket to another.

Two important types:

### Same-Region Replication

```text
Bucket A
   ↓
Bucket B

Same AWS Region
```

### Cross-Region Replication

```text
us-east-1
Bucket A
   ↓
   ↓
Bucket B
   ↓
us-west-2
```

Uses include:

* Disaster recovery
* Data redundancy
* Geographic requirements
* Keeping data in another Region

### Important

For S3 Replication, **Versioning must be enabled on both source and destination buckets**.

---

# 15. S3 Lifecycle vs Replication vs Versioning

These three are easy to confuse.

| Feature         | Purpose                                                |
| --------------- | ------------------------------------------------------ |
| **Versioning**  | Keep multiple versions of an object                    |
| **Replication** | Copy objects to another bucket                         |
| **Lifecycle**   | Automatically transition/delete objects based on rules |

Think:

```text
Versioning
"Keep history."

Replication
"Keep another copy."

Lifecycle
"Manage it automatically as it ages."
```

---

# 16. Snow Family

The **AWS Snow Family** is used for **large-scale data transfer and edge computing**, especially when moving huge amounts of data over the network isn't practical.

Instead of:

```text
Data Center
     ↓
 Internet
     ↓
     S3
```

you can use:

```text
Data Center
     ↓
Snow Device
     ↓
Physical transportation
     ↓
AWS
     ↓
S3
```

Main concepts:

* **Snowcone** → small/portable
* **Snowball** → large-scale data transfer/edge workloads
* **Snowmobile** → extremely large-scale data transfer

### Mental model

> **Snow Family = physically move/process massive amounts of data.**

---

# 17. AWS Storage Gateway

Storage Gateway connects **on-premises applications/infrastructure with AWS storage**.

```text
On-Premises
     ↓
Storage Gateway
     ↓
AWS Storage
```

It is useful when an organization wants to use AWS storage while maintaining existing storage workflows.

### Main gateway types

```text
Storage Gateway
│
├── S3 File Gateway
│
├── Volume Gateway
│
└── Tape Gateway
```

### S3 File Gateway

Makes S3 accessible to on-premises applications through a **file interface** such as NFS/SMB.

```text
On-prem app
     ↓
File share
     ↓
S3 File Gateway
     ↓
S3
```

Mental model:

> **S3 File Gateway = make S3 look like a file share to an on-premises application.**

---

# 18. Snow Family vs Storage Gateway

Very important distinction:

### Snow Family

```text
"I need to MOVE huge amounts of data."
```

### Storage Gateway

```text
"I need to CONNECT my on-premises environment
to AWS storage."
```

---

# 19. S3 in Your AI Backend

This is where everything comes together.

Suppose you're building a **FastAPI RAG application**.

A user uploads a PDF:

```text
User
 ↓
FastAPI
 ↓
S3
 ↓
PDF stored
```

Your database might store:

```text
PostgreSQL / RDS
        ↓
filename
user_id
S3 object key
upload date
metadata
```

Then your processing pipeline can retrieve the PDF from S3:

```text
S3
 ↓
PDF
 ↓
Text extraction
 ↓
Chunking
 ↓
Embeddings
 ↓
Vector DB
```

So you could have:

```text
                    ┌──→ RDS
                    │
User → ALB → FastAPI ├──→ S3
                    │
                    └──→ Vector DB
```

### The core mental model

```text
EC2
→ Run the application

EBS
→ Disk for the EC2 server

RDS
→ Store structured application data

S3
→ Store files/objects

ALB
→ Distribute requests

ASG
→ Manage number of EC2 instances
```

---

# 🧠 S3 Cheat Sheet

If you come back to these notes months later, remember this:

```text
S3
│
├── Bucket
│    └── Container for objects
│
├── Object
│    └── Actual stored data
│
├── Bucket Policy
│    └── Resource-based access control
│
├── Pre-signed URL
│    └── Temporary access to private object
│
├── Static Website
│    └── Host HTML/CSS/JS/etc.
│
├── Storage Classes
│    ├── Standard
│    ├── Standard-IA
│    ├── One Zone-IA
│    └── Glacier
│
├── Lifecycle
│    └── Automatically transition/delete objects
│
├── Versioning
│    └── Keep object history
│
├── Replication
│    └── Copy objects to another bucket
│
├── Snow Family
│    └── Physically move/process huge datasets
│
└── Storage Gateway
     └── Connect on-prem storage/apps to AWS
```

### One-line definition to remember

> **Amazon S3 is AWS's highly scalable object-storage service used to store and manage files/data as objects inside buckets, with features for access control, lifecycle management, versioning, replication, archival, and integration with applications and on-premises environments.**
