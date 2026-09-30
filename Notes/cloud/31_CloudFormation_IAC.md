# AWS CloudFormation

**AWS CloudFormation is AWS's native Infrastructure as Code (IaC) service.**

Its job is:

> **You describe the AWS infrastructure you want in a template, and CloudFormation creates and manages those resources for you.**

---

## 1. The problem CloudFormation solves

Imagine you want to deploy your FastAPI application.

You might need:

```text
VPC
 ├── Subnets
 ├── Security Groups
 ├── EC2
 ├── ALB
 ├── RDS
 └── IAM Role
```

Without IaC, you'd manually configure all of these through the AWS Console.

That's slow and error-prone.

With CloudFormation:

```text
CloudFormation Template
        │
        ▼
   CloudFormation
        │
        ▼
       AWS
        │
 ┌──────┼─────────────┐
 ▼      ▼      ▼      ▼
EC2    RDS    ALB    VPC
```

You describe the desired infrastructure, and CloudFormation provisions it.

---

# 2. What is a CloudFormation Template?

A **template** is the file where you describe your infrastructure.

CloudFormation templates are commonly written in:

* **YAML**
* **JSON**

Since you've already learned YAML, you'll find the syntax much easier.

A very simplified example:

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Resources:

  MyBucket:
    Type: AWS::S3::Bucket
```

This says:

> Create an S3 bucket.

CloudFormation reads the template and creates the resource.

---

# 3. Resources

The most important section is:

```yaml
Resources:
```

This is where you define the AWS resources you want.

For example:

```yaml
Resources:

  MyBucket:
    Type: AWS::S3::Bucket
```

Here:

```text
MyBucket
   │
   └── Logical name you chose

Type:
AWS::S3::Bucket
   │
   └── AWS resource type
```

You could also define:

```yaml
Resources:

  MyBucket:
    Type: AWS::S3::Bucket

  MyInstance:
    Type: AWS::EC2::Instance

  MyDatabase:
    Type: AWS::RDS::DBInstance
```

Now CloudFormation knows you want:

```text
S3
EC2
RDS
```

---

# 4. Stack — VERY IMPORTANT

This is one of the most important CloudFormation concepts.

A **Stack** is a collection of AWS resources managed together by CloudFormation.

Think:

```text
CloudFormation
      │
      ├── Stack: MyApplication
      │      ├── VPC
      │      ├── EC2
      │      ├── ALB
      │      └── RDS
      │
      └── Stack: MyWebsite
             ├── S3
             └── CloudFront
```

The **template** describes what you want.

The **stack** is the actual deployed collection of resources.

### Mental model

```text
Template
   │
   │ "Create these resources"
   ▼
CloudFormation
   │
   ▼
Stack
   │
   ├── EC2
   ├── S3
   ├── RDS
   └── ...
```

---

# 5. Why call it a Stack?

Because your infrastructure is treated as one logical unit.

Suppose your stack contains:

```text
MyApplication
├── EC2
├── Security Group
├── ALB
└── S3
```

You can tell CloudFormation:

> Update my application stack.

CloudFormation determines what needs to change.

You can also delete the stack, and CloudFormation can remove the resources it manages according to their deletion behavior.

This is much easier than manually tracking dozens of AWS resources.

---

# 6. CloudFormation is declarative

This is a **very important IaC concept**.

You generally tell CloudFormation:

> **WHAT infrastructure I want.**

You don't normally write every individual API call saying:

```text
Create VPC
Then create subnet
Then create security group
Then create EC2
...
```

Instead:

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
```

You're describing the **desired state**.

That's called **declarative infrastructure**.

Compare:

```text
Imperative:
"Do these steps."

Declarative:
"I want this final state."
```

CloudFormation figures out the operations necessary to get there.

---

# 7. CloudFormation can understand dependencies

Suppose your application needs:

```text
VPC
 ↓
Subnet
 ↓
EC2
```

CloudFormation can determine resource dependencies and create resources in the appropriate order.

You can also explicitly define dependencies when necessary.

This becomes extremely useful when infrastructure gets complicated.

---

# 8. CloudFormation updates

This is where it becomes really powerful.

Imagine your template says:

```yaml
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: t3.micro
```

Later you change it to:

```yaml
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: t3.small
```

You update the stack.

Conceptually:

```text
Old Template
    │
    ▼
CloudFormation
    │
    │ compare desired configuration
    ▼
Existing Stack
    │
    ▼
Apply required changes
```

You don't have to manually find the EC2 instance and change everything yourself.

---

# 9. CloudFormation vs Terraform

You'll definitely encounter this comparison.

|                 | CloudFormation                  | Terraform               |
| --------------- | ------------------------------- | ----------------------- |
| Created by      | AWS                             | HashiCorp               |
| Main purpose    | IaC                             | IaC                     |
| AWS support     | Excellent/native                | Excellent               |
| Other clouds    | Limited compared with Terraform | Strong multi-cloud      |
| Templates       | YAML/JSON                       | HCL                     |
| AWS integration | Native                          | Provider-based          |
| State           | AWS manages stack state         | Terraform manages state |

The biggest mental distinction:

```text
CloudFormation
      ↓
AWS-native IaC

Terraform
      ↓
Cloud-agnostic IaC tool
```

For your AWS learning, **CloudFormation is important to understand**, but you don't need to become an expert in it before learning Terraform.

---

# 10. CloudFormation + Git

Now connect this with what you've already learned.

Instead of having infrastructure configured only inside your AWS account:

```text
AWS Console
   ↓
Manually created infrastructure
```

you can have:

```text
GitHub
   │
   └── infrastructure.yaml
             │
             ▼
       CloudFormation
             │
             ▼
            AWS
```

Now your infrastructure configuration can be:

* version controlled
* reviewed
* reproduced
* modified
* automated

That's the real **DevOps/IaC mindset**.

---

# 11. One thing you should NOT confuse

CloudFormation **doesn't replace AWS**.

It's a management/automation service.

Think:

```text
AWS
│
├── EC2       → compute
├── S3        → storage
├── RDS       → database
├── Lambda    → serverless compute
│
└── CloudFormation
             ↓
      creates/manages
      those resources
```

CloudFormation itself isn't your EC2 server or database.

It's the **orchestrator for infrastructure defined in your template**.

---

## 🧠 The whole concept

Keep this mental model:

```text
                YOU
                 │
                 ▼
        CloudFormation Template
              (YAML/JSON)
                 │
                 ▼
          AWS CloudFormation
                 │
              creates
                 │
                 ▼
              STACK
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
      EC2       RDS       S3
```

### In one sentence:

**CloudFormation = AWS's native IaC service that uses templates to declaratively create, update, and manage collections of AWS resources called stacks.**

For your learning, the next concepts should be **Template → Resources → Properties → Parameters → Outputs → Stack → Change Sets → Drift Detection**, and then we'll build a **small CloudFormation template ourselves** rather than jumping straight into a huge AWS architecture.
