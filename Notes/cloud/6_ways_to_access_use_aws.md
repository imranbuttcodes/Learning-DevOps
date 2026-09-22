AWS can be accessed/controlled through several interfaces:

```text
                         AWS
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
   Web Console          CLI              Programmatic
    (Browser)          Commands             Access
                                             │
                              ┌──────────────┼──────────────┐
                              ▼              ▼              ▼
                           SDKs          APIs           Scripts
```

### 1. AWS Management Console — Browser 🌐

This is what you've been using.

You open the AWS website and click:

```text
EC2 → Launch instance
S3 → Create bucket
IAM → Create user
```

Good for:

* Learning
* Exploring services
* One-off administrative tasks
* Visual management

---

### 2. AWS CLI — Terminal 💻

The **AWS CLI** lets you interact with AWS from a terminal.

For example:

```bash
aws s3 ls
```

or:

```bash
aws ec2 describe-instances
```

You can run it from:

* Your **local computer**
* An AWS-hosted terminal such as **CloudShell**

So your "browser's CLI" is most likely **AWS CloudShell**.

```text
Browser
 ├── AWS Console
 └── CloudShell
       ↓
    AWS CLI
```

Your local computer is:

```text
Ubuntu
  ↓
Terminal
  ↓
AWS CLI
  ↓
AWS
```

---

### 3. Scripts 🤖

You can automate AWS operations using scripts.

For example, a Bash/Python script could:

```text
Create resource
      ↓
Configure resource
      ↓
Deploy application
      ↓
Check status
      ↓
Delete resource
```

Instead of manually clicking through the Console every time.

The script might internally use the **AWS CLI** or an **SDK**.

---

### 4. SDKs — Software Development Kits 👨‍💻

An SDK allows your **application code** to interact with AWS.

For example, your Python FastAPI application could use the AWS Python SDK (**boto3**) to interact with S3.

Conceptually:

```text
FastAPI / Python
       ↓
     SDK
       ↓
    AWS API
       ↓
      S3
```

This is different from manually running:

```bash
aws s3 ls
```

Your **application itself** is making the AWS request.

---

### 5. AWS APIs

At the lowest level, AWS services expose **APIs**.

Conceptually:

```text
Your Application
       ↓
      SDK
       ↓
   AWS API
       ↓
      S3
```

The SDK is essentially a convenient programming interface around the underlying service APIs.

---

## 🧠 The important distinction

Don't memorize these as five completely separate things.

Think:

```text
                   AWS
                    ▲
                    │
       ┌────────────┼────────────┐
       │            │            │
    Console        CLI          SDK
       │            │            │
    Browser      Terminal      Program
                                │
                                ▼
                              API
```

And **scripts** are automation that can use either CLI or SDK:

```text
Bash/Python Script
       │
   ┌───┴────┐
   ▼        ▼
  CLI      SDK
   │        │
   └────┬───┘
        ▼
       AWS
```

### For your DevOps path

You should eventually be comfortable with:

**Console → AWS CLI → Python SDK → automation**

You don't need to manually learn every AWS API endpoint. In practice, you'll commonly use the **Console for exploration**, **CLI for administration/automation**, and **SDKs when your applications need to interact with AWS programmatically**.
