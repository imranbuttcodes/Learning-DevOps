# AWS EBS (Elastic Block Store) — Complete Learning Notes

## 1. What is EBS?

**Amazon Elastic Block Store (EBS)** is AWS's persistent **block storage** service for EC2.

Mental model:

```text
EC2 = Computer / Compute
EBS = Virtual Hard Drive / Storage
````

An EBS volume behaves like a virtual disk that an EC2 instance can use for its operating system, applications, and data.

---

## 2. What does "Block Storage" mean?

Block storage stores data in addressable blocks.

Conceptually:

```text
EBS Volume
┌─────────────────────────────────┐
│ Block │ Block │ Block │ Block │
│   1   │   2   │   3   │   4   │
└─────────────────────────────────┘
          ↓
       Filesystem
          ↓
    Files / Directories
```

The operating system places a filesystem on the EBS volume and uses it like a disk.

---

# 3. Root EBS Volume

When launching a normal EBS-backed EC2 instance, the root EBS volume contains the operating system.

```text
EC2 Instance
│
└── Root EBS Volume
      ├── Amazon Linux
      ├── system files
      ├── installed software
      └── application files
```

If the root volume is removed, the instance has no normal boot disk.

---

# 4. EBS is Persistent

EBS data generally survives an EC2 **stop/start** cycle.

```text
EC2 Running
    ↓
   Stop
    ↓
EC2 Stopped
    ↓
   Start
    ↓
EC2 Running

EBS data remains
```

However, what happens to an EBS volume when an EC2 instance is **terminated** depends on its `DeleteOnTermination` setting.

For many root volumes, `DeleteOnTermination` is enabled by default.

A separately created data volume can be configured to survive instance termination.

---

# 5. EBS Volume vs Instance Store

### EBS

* Persistent
* Network-attached storage
* Can be detached and attached to another compatible EC2 instance
* Supports snapshots

### Instance Store

* Physically attached to the host
* Very fast for supported instance types
* Ephemeral
* Data can be lost when the instance is stopped/terminated or otherwise loses its host

Mental model:

```text
EBS             → Persistent disk
Instance Store  → Temporary local disk
```

---

# 6. EBS Volume Types

Common EBS volume types include:

| Type  | General idea              |
| ----- | ------------------------- |
| `gp3` | General-purpose SSD       |
| `gp2` | Older general-purpose SSD |
| `io2` | Provisioned IOPS SSD      |
| `st1` | Throughput-optimized HDD  |
| `sc1` | Cold HDD                  |

For normal application/server workloads, **gp3** is an important type to understand.

---

# 7. EBS is AZ-Specific

An EBS volume belongs to a specific **Availability Zone**.

Example:

```text
Region: us-east-1

AZ-A
 └── EBS Volume A

AZ-B
 └── EC2 Instance B
```

You cannot simply attach the EBS volume in AZ-A directly to an EC2 instance in AZ-B.

A common solution is:

```text
EBS Volume
    ↓
Snapshot
    ↓
Create new EBS volume
in another AZ
```

---

# 8. Attaching EBS Volumes

Typical workflow:

```text
Create EBS Volume
       ↓
Choose Availability Zone
       ↓
Attach to EC2
       ↓
Format filesystem
       ↓
Mount
       ↓
Use as storage
```

Normally an EBS volume is attached to one EC2 instance at a time.

### Exception: Multi-Attach

Some supported EBS configurations, such as supported `io1`/`io2` setups, can allow a volume to be attached to multiple instances.

This is an advanced feature and requires appropriate workload/application design.

---

# 9. EBS Snapshots

A **snapshot** is a point-in-time backup of an EBS volume.

```text
EBS Volume
    │
    │ Create Snapshot
    ↓
Snapshot
```

The snapshot itself is not normally attached directly to EC2.

Instead:

```text
Snapshot
    ↓
Create EBS Volume
    ↓
Attach to EC2
```

---

# 10. What is included in a Snapshot?

It depends on which EBS volume you snapshot.

## Root volume snapshot

If you snapshot the root EBS volume:

```text
Root EBS
├── Operating System
├── system files
├── installed software
└── user/application data
```

The snapshot represents that volume's contents.

## Data volume snapshot

If you snapshot a separate data volume:

```text
Data EBS
├── application data
├── database files
└── other files
```

It does **not** contain the operating system from the separate root volume.

---

# 11. Snapshots are Not Automatically Created

AWS does not automatically create ordinary EBS snapshots just because an EBS volume exists.

You can:

* Create snapshots manually
* Configure automation
* Use AWS Backup
* Use EBS Lifecycle Manager

EBS durability itself is not the same thing as having a user-managed backup.

---

# 12. Incremental Snapshots

EBS snapshots use an incremental approach after the initial snapshot.

Conceptually:

```text
Snapshot 1
└── Initial snapshot data

Snapshot 2
└── New/changed blocks since previous snapshot

Snapshot 3
└── Further changed blocks
```

AWS manages the underlying snapshot storage.

---

# 13. Snapshot vs EBS Volume vs AMI

These are easy to confuse.

```text
EBS Volume
    = Actual virtual disk

Snapshot
    = Point-in-time backup of an EBS volume

AMI
    = Image/template used to launch EC2 instances
```

Mental model:

```text
EBS Volume
    │
    └── Snapshot
          = Backup / checkpoint

AMI
    └── EC2 launch image/template
```

---

# 14. Copying Snapshots Across Regions

EBS snapshots are Region-specific.

You can copy a snapshot to another AWS Region.

```text
us-east-1
    │
    │ Copy Snapshot
    ↓
us-west-2
```

The copied snapshot is a separate snapshot in the target Region.

Then:

```text
Copied Snapshot
      ↓
Create EBS Volume
      ↓
Choose AZ in target Region
      ↓
Attach to EC2
```

### Why copy snapshots across Regions?

A major use case is **disaster recovery**.

```text
Primary Region
     │
     │ Snapshot Copy
     ↓
Backup Region
```

If the primary Region becomes unavailable, the backup snapshot exists in another Region.

Cross-Region copies can involve additional storage/data-transfer costs and, for encrypted snapshots, appropriate KMS permissions/key considerations.

---

# 15. EBS Lifecycle Manager (DLM)

**EBS Lifecycle Manager (DLM)** automates the creation and retention of EBS snapshots according to a policy.

Without DLM:

```text
Create snapshot
     ↓
Create snapshot
     ↓
Delete old snapshot
     ↓
Create snapshot
     ↓
...
```

With DLM:

```text
EBS Volume
     ↓
DLM Policy
     ↓
Automatic Snapshots
     ↓
Retention Rules
```

DLM can automatically create snapshots and manage their lifecycle.

---

# 16. DLM Schedule

A DLM policy can contain one or more schedules.

Example:

```text
Frequency: Daily
Every: 12 hours
```

This means snapshots are scheduled every 12 hours.

Conceptually:

```text
00:00 → Snapshot
12:00 → Snapshot
00:00 → Snapshot
12:00 → Snapshot
...
```

---

# 17. DLM Retention Type

DLM provides retention options such as **Count** and **Age**.

## Count

Count means:

> Keep a specific number of snapshots.

Example:

```text
Retention type = Count
Keep = 7
```

The policy keeps the configured number of snapshots.

Mental shortcut:

```text
Count = HOW MANY?
```

---

## Age

Age means:

> Keep each snapshot for a specified amount of time after creation.

Example:

```text
Retention type = Age
Keep = 30 days
```

Each snapshot gets its own retention period:

```text
Snapshot created
      ↓
     30 days
      ↓
Expires from the configured tier
```

Mental shortcut:

```text
Age = HOW OLD?
```

### Important distinction

```text
Count + Keep 7
→ Keep 7 snapshots

Age + Keep 30 days
→ Keep snapshots for 30 days
```

So:

**Retention type = How AWS measures retention**

**Keep = The actual retention limit**

---

# 18. DLM "Standard Tier"

EBS snapshots can use different storage tiers.

The **Standard tier** is the normal snapshot storage tier.

When DLM says:

```text
Expire from standard tier
```

it means the snapshot's lifecycle in the standard storage tier is being configured.

For example:

```text
Retention type: Age
Keep: 30 days
Expire from standard tier: 30 days after creation
```

---

# 19. DLM Advanced Settings

DLM schedules provide several optional advanced settings.

## A. Tagging

Automatically applies tags to snapshots created by the schedule.

Example:

```text
Backup = Daily
Environment = Production
Owner = Imran
```

Useful for organizing and identifying snapshots.

Important:

> Tags configured for the schedule are not automatically applied to cross-Region copies created by the schedule.

---

## B. Snapshot Archiving

Allows snapshots to automatically move from the **Standard storage tier** to the **Archive storage tier**.

```text
Standard Tier
      ↓
Archive Tier
```

Useful for backups that need to be retained for a long time but are rarely accessed.

---

## C. Fast Snapshot Restore (FSR)

Fast Snapshot Restore prepares a snapshot so that volumes created from it can immediately deliver their provisioned performance.

Useful when rapid restoration and immediate performance are important.

It is an advanced feature and can incur additional cost.

---

## D. Cross-Region Copy

Automatically copies snapshots created by the schedule to additional AWS Regions.

Example:

```text
us-east-1
    │
    ├── Original Snapshot
    │
    └── Cross-Region Copy
             ↓
          us-west-2
```

Useful for disaster recovery.

---

## E. Cross-Account Sharing

Allows snapshots to be shared with another AWS account.

```text
AWS Account A
      │
      │ Share Snapshot
      ↓
AWS Account B
```

Useful for organizational, backup, migration, or multi-account architectures.

---

# 20. Complete EBS Mental Model

```text
                         EC2
                          │
                 ┌────────┴────────┐
                 │                 │
             Root EBS          Data EBS
                 │                 │
                 └────────┬────────┘
                          │
                       Snapshot
                          │
                     ┌────┴────┐
                     │         │
                  Manual      DLM
                  Backup      Policy
                               │
                    ┌──────────┼──────────┐
                    │          │          │
                 Schedule  Retention   Advanced
                    │          │          │
                Every 12h   Count/Age   Archive
                                        FSR
                                        Cross-Region
                                        Sharing
```

---

# 21. EBS Final Cheat Sheet

| Concept               | Meaning                                                         |
| --------------------- | --------------------------------------------------------------- |
| EC2                   | Compute / virtual server                                        |
| EBS                   | Persistent block storage                                        |
| EBS Volume            | Virtual disk                                                    |
| Root Volume           | EBS volume containing the OS                                    |
| Snapshot              | Point-in-time backup of an EBS volume                           |
| DLM                   | Automates snapshot lifecycle                                    |
| Count                 | Retention based on number of snapshots                          |
| Age                   | Retention based on time                                         |
| Standard Tier         | Normal snapshot storage                                         |
| Archive Tier          | Long-term snapshot storage                                      |
| FSR                   | Faster/immediate performance for volumes created from snapshots |
| Cross-Region Copy     | Copy snapshot to another Region                                 |
| Cross-Account Sharing | Share snapshot with another AWS account                         |
| gp3                   | General-purpose SSD                                             |
| AZ-specific           | EBS volume belongs to one Availability Zone                     |
| Stop/Start            | EBS data generally remains                                      |
| Termination           | EBS deletion depends on `DeleteOnTermination`                   |

---

# 22. Most Important Mental Model

Remember these four lines:

```text
EC2      = Computer
EBS      = Hard Drive
Snapshot = Backup
DLM      = Automatic Backup/Lifecycle Manager
```

And:

```text
EBS Volume → AZ-specific
Snapshot   → Region-specific
Snapshot Copy → Separate snapshot in target Region

Count = HOW MANY?
Age   = HOW LONG?
```