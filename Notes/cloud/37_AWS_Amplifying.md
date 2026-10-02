Let's learn **AWS Amplify Hosting** like a DevOps/AWS concept first.

# AWS Amplify Hosting — Explanation Guide

## 1. First: What problem does Amplify solve?

Suppose you've built a React application:

```text
React Application
        │
        ▼
     GitHub
```

Now you need to make it available on the Internet.

Traditionally, you could manually build a whole hosting pipeline:

```text
GitHub
   ↓
Build React
   ↓
Upload files to S3
   ↓
Configure CloudFront
   ↓
Configure DNS
   ↓
Configure HTTPS
   ↓
Deploy
```

That's a lot of infrastructure for simply hosting a frontend.

**Amplify Hosting simplifies this workflow.**

Conceptually:

```text
GitHub
   │
   ▼
AWS Amplify
   │
   ├── Build
   ├── Deploy
   ├── Host
   ├── HTTPS
   └── CDN delivery
```

So the fundamental idea is:

> **Amplify Hosting is an AWS-managed hosting and deployment platform for web applications.**

---

# 2. What can you host?

Amplify Hosting is primarily useful for **web applications**, including frameworks such as:

* React
* Next.js
* Vue
* Angular
* Gatsby
* static HTML/CSS/JS

For your learning path, the important example is:

```text
React
   ↓
Amplify Hosting
```

You don't need to think about EC2 for this.

You aren't renting a server and managing Linux yourself.

---

# 3. The most important mental model

Think of Amplify as sitting between your **Git repository** and your **users**.

```text
             DEVELOPMENT
                  │
                  ▼
               GitHub
                  │
             git push
                  │
                  ▼
          ┌───────────────┐
          │    Amplify    │
          │    Hosting    │
          └───────┬───────┘
                  │
             Build + Deploy
                  │
                  ▼
              Your App
                  │
                  ▼
                Users
```

This is the core idea.

---

# 4. GitHub connection

Why does Amplify connect to GitHub?

Because your application source code lives there.

For example:

```text
GitHub
└── my-react-app
    ├── src/
    ├── public/
    ├── package.json
    └── ...
```

You connect:

```text
GitHub Repository
        │
        ▼
    Amplify App
```

Then Amplify watches the repository/branch you've connected.

For example:

```text
GitHub
│
└── main
      │
      ▼
   Amplify
```

---

# 5. What happens when you `git push`?

This is the important DevOps part.

You change your React application:

```text
<h1>Hello</h1>
```

to:

```text
<h1>Hello from my new app</h1>
```

Then:

```text
git add .
git commit
git push
```

Now:

```text
Developer
    │
    │ git push
    ▼
 GitHub
    │
    │ detects new commit
    ▼
 Amplify
    │
    ▼
 Build
    │
    ▼
 Deploy
    │
    ▼
 Live application
```

So Amplify gives you **continuous deployment** for the connected frontend.

You don't manually upload the new files every time.

---

# 6. What does "Build" actually mean?

This is where your Node/React knowledge becomes important.

Your GitHub repository contains **source code**.

For example:

```text
src/
package.json
vite.config.js
```

A browser doesn't necessarily run that source directly.

Amplify needs to execute your project's build process.

For a Vite React application:

```text
npm install
      ↓
npm run build
      ↓
dist/
```

The `dist/` directory contains the production-ready frontend assets.

Conceptually:

```text
SOURCE CODE
     │
     │ Build
     ▼
PRODUCTION FILES
     │
     ▼
   Hosting
```

That's why Amplify needs **build settings**.

---

# 7. Build settings

Build settings tell Amplify:

> "How do I turn this repository into something I can deploy?"

For example:

```yaml
preBuild:
  npm ci

build:
  npm run build

output:
  dist/
```

So Amplify needs to know:

### What dependencies should I install?

```text
npm ci
```

### What command builds the application?

```text
npm run build
```

### Where are the resulting files?

```text
dist/
```

This is the purpose of `amplify.yml`.

---

# 8. What is `amplify.yml`?

It's a **build specification/configuration file**.

Conceptually:

```text
amplify.yml
     │
     ├── How to install dependencies
     ├── How to build
     ├── What files to deploy
     └── What to cache
```

So:

```text
GitHub
   │
   ├── source code
   ├── package.json
   └── amplify.yml
             │
             ▼
          Amplify
```

The file isn't your application.

It's instructions for **how Amplify should build your application**.

---

# 9. Automatic deployment = CI/CD concept

You've already learned GitHub Actions.

Compare them.

### GitHub Actions

You might have:

```text
GitHub
  ↓
GitHub Actions
  ↓
Tests
  ↓
Build
  ↓
Docker
  ↓
Deployment
```

### Amplify Hosting

For a frontend:

```text
GitHub
  ↓
Amplify
  ↓
Build
  ↓
Deploy
  ↓
Hosting
```

Amplify is therefore giving you a more **AWS-integrated frontend deployment workflow** rather than making you build every piece yourself.

---

# 10. What happens after deployment?

Now your application needs to be delivered to users.

This is where **CloudFront** enters the picture.

You already learned CloudFront separately.

CloudFront is a **CDN**.

Its job is essentially:

> Deliver content from locations geographically closer to users and cache appropriate content.

Amplify Hosting uses AWS's CDN infrastructure, including CloudFront, to deliver hosted web content globally.

So conceptually:

```text
             Amplify
                │
                │ hosting
                ▼
           CloudFront
          /     |     \
         /      |      \
      Edge     Edge    Edge
       │        │       │
       ▼        ▼       ▼
     User     User    User
```

---

# 11. So is Amplify = CloudFront?

**No.**

This distinction is critical.

### CloudFront

CloudFront is primarily:

```text
CDN
+
content delivery/caching
```

### Amplify Hosting

Amplify provides a broader workflow:

```text
Source integration
       +
Build
       +
Deployment
       +
Hosting
       +
Domain/HTTPS integration
       +
CDN-backed delivery
```

So:

```text
Amplify
   │
   ├── Source integration
   ├── Build
   ├── Deployment
   ├── Hosting
   └── CloudFront-backed delivery
```

---

# 12. Compare with your S3 + CloudFront lab

You previously learned:

```text
S3
 │
 │ origin
 ▼
CloudFront
 │
 ▼
Users
```

You manually configured those components.

With Amplify:

```text
GitHub
  │
  ▼
Amplify
  │
  ├── Build
  ├── Deploy
  └── Hosting
         │
         ▼
     CloudFront
         │
         ▼
       Users
```

**Amplify abstracts a lot of the infrastructure management.**

That's the important difference.

---

# 13. Custom domain

Initially Amplify gives you an AWS-hosted domain.

Conceptually:

```text
https://something.amplifyapp.com
```

But you may own:

```text
example.com
```

You want:

```text
https://example.com
```

So you connect your domain to the Amplify application.

Architecture becomes:

```text
User
 │
 │ example.com
 ▼
DNS
 │
 ▼
Amplify
 │
 ▼
CloudFront
 │
 ▼
React Application
```

---

# 14. Where does Route 53 fit?

Route 53 is your **DNS service**.

Its job is different from Amplify.

```text
Route 53
    │
    │ DNS
    ▼
"Where should example.com go?"
```

Amplify is concerned with:

```text
"How do I build, deploy and host this application?"
```

So:

```text
Route 53
   │
   │ DNS
   ▼
Amplify / CloudFront
   │
   ▼
Application
```

They complement each other.

---

# 15. HTTPS

For a production website you want:

```text
https://example.com
```

rather than:

```text
http://example.com
```

Amplify's custom-domain integration can handle the certificate/HTTPS side using AWS certificate infrastructure.

So you don't need to manually configure an Nginx server and Let's Encrypt just to get HTTPS for a basic Amplify-hosted frontend.

---

# 16. What about backend?

This is where you need to be careful.

**Amplify Hosting does not mean your entire backend must run inside Amplify.**

You can have:

```text
                 Internet
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
       Frontend             Backend
       Amplify              FastAPI
          │                   │
          │                   ▼
          │                  RDS
          │
          └──── API requests ──→
```

For your career stack, this is a very realistic model:

```text
React
  │
  │ HTTPS API requests
  ▼
FastAPI
  │
  ├── RDS
  ├── DynamoDB
  ├── S3
  └── AI APIs
```

Amplify can handle the **frontend hosting/deployment** while you manage the FastAPI backend separately.

---

# 17. Frontend vs backend responsibility

This distinction is worth memorizing.

### Amplify Hosting

```text
React
Vue
Next.js
Static assets
Frontend deployment
Frontend hosting
```

### EC2 / ECS / Lambda / etc.

Potentially:

```text
FastAPI
Node.js API
AI backend
Background workers
Business logic
```

For example:

```text
                    USER
                      │
                      ▼
              React Frontend
                      │
                 Amplify
                      │
                 CloudFront
                      │
                      │ API request
                      ▼
                 FastAPI API
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
            RDS    DynamoDB    S3
```

---

# 18. Why would someone use Amplify?

Imagine you are building:

> React frontend + FastAPI backend

You could manually build:

```text
S3
CloudFront
Route 53
ACM
CI/CD
Build system
Deployment scripts
```

Or for the frontend you can use:

```text
GitHub
   ↓
Amplify
```

and let Amplify handle much of that frontend hosting workflow.

So the tradeoff is essentially:

```text
Manual AWS infrastructure
        ↓
More control
More configuration
More responsibility


Amplify
        ↓
Higher abstraction
Less configuration
Faster application deployment
```

---

# 19. Amplify vs EC2

This is another common confusion.

### EC2

You're essentially saying:

> "Give me a virtual server."

```text
EC2
 │
 ├── Linux
 ├── packages
 ├── application
 ├── Nginx
 ├── Docker
 └── YOU manage it
```

### Amplify Hosting

You're essentially saying:

> "Here's my web application repository. Build and host it."

```text
GitHub
  │
  ▼
Amplify
  │
  ├── Build
  ├── Deploy
  └── Host
```

So:

**EC2 = infrastructure/server-oriented**

**Amplify Hosting = application/frontend-oriented**

---

# 20. The complete mental model

This is the one I'd remember:

```text
                         DEVELOPER
                             │
                         git push
                             │
                             ▼
                          GitHub
                             │
                             ▼
                    ┌─────────────────┐
                    │ AWS Amplify     │
                    │                 │
                    │ Build           │
                    │ Deploy          │
                    │ Host            │
                    └────────┬────────┘
                             │
                             ▼
                       CloudFront
                          CDN
                             │
                    ┌────────┼────────┐
                    ▼        ▼        ▼
                  Edge     Edge     Edge
                    │        │        │
                    └────────┼────────┘
                             ▼
                           Users
```

With your own domain:

```text
User
 │
 │ example.com
 ▼
Route 53
 │
 ▼
Amplify
 │
 ▼
CloudFront
 │
 ▼
React
```

And with your FastAPI backend:

```text
                         USER
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
      React Frontend                FastAPI API
         Amplify                        │
           │                            │
      CloudFront                        ▼
           │                           RDS
           │
           └──── HTTPS API calls ───────┘
```

## The five things you should now know

| Concept                  | What it means                                               |
| ------------------------ | ----------------------------------------------------------- |
| **Amplify Hosting**      | Managed frontend/web-app hosting and deployment             |
| **GitHub connection**    | Amplify gets your source code and watches selected branches |
| **Build settings**       | Instructions for turning source code into deployable files  |
| **Automatic deployment** | A new Git commit can trigger a new build/deployment         |
| **CloudFront**           | CDN used for global delivery of hosted content              |
| **Custom domain**        | Lets users access the app through your own domain           |
| **Route 53**             | DNS service that maps your domain to the hosted application |

### The one-line definition

> **AWS Amplify Hosting is a managed AWS platform that connects your web application's source repository to an automated build/deployment pipeline and globally distributed hosting, with CloudFront handling content delivery.**

That is the level of understanding I'd want you to have before touching the Amplify console.
