**“Common cloud services”** basically means the major categories of resources you commonly use from cloud providers like AWS, Azure, and Google Cloud.

Think of cloud computing as renting different building blocks for your application:

| Category                   | What it provides                    | AWS example              |
| -------------------------- | ----------------------------------- | ------------------------ |
| 🖥️ **Compute**            | Run applications/code               | EC2                      |
| 📦 **Object Storage**      | Store files, images, PDFs, backups  | S3                       |
| 💾 **Block Storage**       | Disk attached to a server           | EBS                      |
| 🗄️ **Databases**          | Store structured application data   | RDS                      |
| 🌐 **Networking**          | Connect and isolate resources       | VPC                      |
| 🔐 **Identity & Security** | Users, permissions, authentication  | IAM                      |
| ⚖️ **Load Balancing**      | Distribute traffic across servers   | ELB/ALB                  |
| 📈 **Auto Scaling**        | Automatically add/remove compute    | EC2 Auto Scaling         |
| 🌍 **DNS**                 | Map domains to services             | Route 53                 |
| 🚀 **CDN**                 | Cache content close to users        | CloudFront               |
| ⚡ **Serverless**           | Run code without managing servers   | Lambda                   |
| 📊 **Monitoring**          | Metrics, logs, alarms               | CloudWatch               |
| 🐳 **Containers**          | Run/manage Docker containers        | ECS / Fargate            |
| 📨 **Messaging**           | Services communicate asynchronously | SQS / SNS                |
| 🔄 **CI/CD**               | Build/test/deploy automatically     | CodePipeline / CodeBuild |
| 🧱 **IaC**                 | Define infrastructure as code       | CloudFormation           |
| 🤖 **AI/ML**               | AI models and ML infrastructure     | Bedrock / SageMaker      |

### The big picture

For an application like your **AI backend**, you might have:

```text
                  Internet
                     │
                     ▼
                Route 53
                  (DNS)
                     │
                     ▼
              CloudFront / ALB
                     │
                     ▼
              ┌─────────────┐
              │    EC2      │
              │ Docker App  │
              └─────────────┘
                 │       │
          ┌──────┘       └──────┐
          ▼                     ▼
       RDS DB                  S3
    application data       PDFs/files
          │
          ▼
      CloudWatch
    logs + metrics
```

And **IAM** controls who/what is allowed to access these resources.

### What you actually need to memorize

Don't try to memorize 100 AWS services. For your DevOps/AI-backend path, get very comfortable with:

**EC2 → S3 → RDS → VPC → IAM → Security Groups → ALB → Auto Scaling → Route 53 → CloudFront → CloudWatch → ECS/ECR**

Then understand **Lambda, SQS/SNS, CloudFormation, and Bedrock** conceptually.

That's enough to give you a very solid understanding of what the cloud actually provides.
