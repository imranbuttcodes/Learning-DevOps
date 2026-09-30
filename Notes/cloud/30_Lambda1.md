# AWS Lambda — Start With the Concept

Before touching the AWS console, let's understand **why Lambda exists**.

## 1. What is Lambda?

**AWS Lambda is a serverless compute service.**

It lets you run code **without managing the underlying server yourself**.

With EC2, you do:

```text
You
 ↓
Create EC2
 ↓
Choose instance type
 ↓
OS
 ↓
SSH
 ↓
Install Python/Node
 ↓
Deploy application
 ↓
Keep server running
```

With Lambda:

```text
You
 ↓
Upload code
 ↓
AWS Lambda
 ↓
AWS runs it when needed
```

You focus mainly on the **function/code**, while AWS manages the underlying compute infrastructure.

---

# 2. What does "serverless" actually mean?

This is important:

> **Serverless does NOT mean there are no servers.**

There are absolutely servers.

AWS owns and operates them.

You simply don't manage those servers directly.

So:

```text
EC2

You → manage server → run application
```

while:

```text
Lambda

You → provide function
AWS → manages infrastructure
```

That's why it's called **serverless**.

---

# 3. Why would we use Lambda?

Imagine you have a tiny piece of code that needs to run whenever something happens.

For example:

> "Whenever someone uploads an image to S3, resize it."

You don't necessarily need an EC2 server running 24/7 just waiting for an upload.

Instead:

```text
User
 ↓
Upload image
 ↓
S3
 ↓
Lambda triggered
 ↓
Resize image
 ↓
Save result
```

Lambda runs the code when needed.

Then the execution ends.

---

# 4. Lambda is based around FUNCTIONS

This is the key mental model.

With a traditional server:

```text
EC2
 ↓
FastAPI
 ├── /users
 ├── /orders
 ├── /products
 └── /payments
```

The server is continuously running your application.

Lambda is more like:

```text
Lambda Function
      ↓
   handler()
      ↓
   execute
      ↓
   finish
```

You create a function that AWS can invoke.

For example conceptually:

```python
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Hello!"
    }
```

Don't worry about the code yet. We're learning the architecture first.

---

# 5. What triggers Lambda?

This is where Lambda becomes really powerful.

Lambda doesn't necessarily sit there waiting for HTTP requests.

Many AWS services/events can **invoke** Lambda.

For example:

```text
                    Lambda
                       ↑
        ┌──────────────┼──────────────┐
        │              │              │
       S3           API Gateway     EventBridge
        │              │              │
   file upload      HTTP request    scheduled event
```

Other possibilities include:

* SQS messages
* DynamoDB Streams
* EventBridge events
* CloudWatch-related events
* API Gateway requests
* S3 events
* Application events

The general pattern is:

```text
EVENT
  ↓
Lambda
  ↓
FUNCTION EXECUTES
  ↓
RESULT
```

---

# 6. Lambda + API Gateway

This is one of the most common architectures.

Suppose you want an API but don't want to run an EC2 server.

You can have:

```text
User
 ↓
API Gateway
 ↓
Lambda
 ↓
DynamoDB
```

For example:

```text
GET /users/123
       ↓
 API Gateway
       ↓
 Lambda
       ↓
 DynamoDB
       ↓
   User data
```

Compare that with your current FastAPI architecture:

```text
User
 ↓
EC2
 ↓
Docker
 ↓
FastAPI
 ↓
RDS MySQL
```

So Lambda can potentially replace the **always-running application server** for certain workloads.

---

# 7. Lambda + DynamoDB

This combination is extremely common.

For example, imagine a serverless user system:

```text
             ┌─────────────┐
             │ API Gateway │
             └──────┬──────┘
                    ↓
               ┌─────────┐
               │ Lambda  │
               └────┬────┘
                    ↓
              ┌───────────┐
              │ DynamoDB  │
              └───────────┘
```

There is no EC2 server in this architecture.

AWS manages the compute and database infrastructure.

This is often called a **serverless architecture**.

---

# 8. Lambda isn't just for APIs

This is another important point.

Lambda can perform background tasks.

### Example: S3

```text
User
 ↓
Upload PDF
 ↓
S3
 ↓
Lambda
 ↓
Process PDF
```

### Example: scheduled task

```text
Every night at 2 AM
        ↓
    Lambda
        ↓
Clean temporary data
```

### Example: queue

```text
Application
    ↓
   SQS
    ↓
 Lambda
    ↓
Process message
```

So Lambda is really:

> **Run this piece of code when this event occurs.**

---

# 9. Lambda vs EC2

This is probably the most important comparison for you.

| EC2                                      | Lambda                             |
| ---------------------------------------- | ---------------------------------- |
| Virtual server                           | Serverless function execution      |
| You manage OS                            | AWS manages infrastructure         |
| Usually continuously running             | Runs when invoked                  |
| You choose instance type                 | AWS manages compute allocation     |
| SSH possible                             | No normal server SSH               |
| Can run arbitrary long-running processes | Designed for bounded executions    |
| You manage server environment            | AWS manages runtime infrastructure |
| Great for traditional servers            | Great for event-driven workloads   |

---

# 10. Your current FastAPI application vs Lambda

You currently have:

```text
Browser
   ↓
EC2 :80
   ↓
Docker
   ↓
FastAPI :8000
   ↓
RDS MySQL :3306
```

A serverless version could look like:

```text
Browser
   ↓
API Gateway
   ↓
Lambda
   ↓
DynamoDB
```

Notice something interesting:

### Current architecture

You manage:

```text
EC2
Docker
FastAPI
```

### Serverless architecture

AWS manages the compute infrastructure and you mainly manage:

```text
Lambda function
API Gateway configuration
DynamoDB
IAM permissions
```

---

# 11. But Lambda isn't automatically better

This is important.

Lambda isn't:

> "EC2 but better."

Different workloads fit different models.

For example, a continuously running application may be more naturally hosted on EC2 or containers.

Lambda is particularly useful when:

* Work is event-driven
* Execution is relatively short-lived
* Traffic varies significantly
* You don't want to manage servers
* You want AWS to handle infrastructure scaling

---

# 12. Lambda and scaling

Imagine your Lambda function receives:

```text
1 request
```

AWS runs the function.

Then:

```text
100 requests
```

AWS can run multiple executions.

Conceptually:

```text
              Lambda
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
   Function   Function   Function
   execution  execution  execution
```

You aren't manually launching three EC2 instances.

AWS handles the underlying execution infrastructure.

---

# 13. Lambda has a runtime

Lambda needs to know what environment should execute your code.

For example, AWS Lambda supports runtimes for languages such as:

```text
Python
Node.js
Java
.NET
Ruby
```

You can also package applications using **container images** in supported Lambda workflows.

So Lambda isn't restricted to one programming language.

---

# 14. Lambda + IAM

And here's where our previous IAM learning becomes important.

Suppose Lambda needs to read from DynamoDB.

Don't put an IAM user's access key inside your Lambda code.

Instead:

```text
Lambda
   ↓
IAM Role
   ↓
Permission:
dynamodb:GetItem
dynamodb:PutItem
   ↓
DynamoDB
```

This is the same principle we discussed with EC2.

**AWS workloads should generally use IAM roles rather than hard-coded IAM user credentials.**

---

# 15. The Lambda mental model

Remember this:

```text
                    EVENT
                      │
                      ↓
                ┌──────────┐
                │  Lambda  │
                │ Function │
                └────┬─────┘
                     │
                     ↓
                  Execute
                     │
                     ↓
                   Result
```

And the event could come from:

```text
S3
API Gateway
SQS
EventBridge
DynamoDB Streams
etc.
```

---

# 16. Where we're going next

I wouldn't jump directly into the AWS console yet.

We should learn Lambda in this order:

```text
1. What Lambda is
        ↓
2. Serverless concept
        ↓
3. Lambda Function
        ↓
4. Handler
        ↓
5. Event & Context
        ↓
6. Invocation
        ↓
7. Triggers
        ↓
8. Lambda execution lifecycle
        ↓
9. Statelessness
        ↓
10. Timeout & memory
        ↓
11. IAM execution role
        ↓
12. Lambda + API Gateway
        ↓
13. Lambda + DynamoDB
        ↓
14. Lambda + S3
        ↓
15. Hands-on
```

The **next concept I'd focus on is `Handler + Event + Context`**, because once those three click, the actual Lambda console and code become much easier to understand.
