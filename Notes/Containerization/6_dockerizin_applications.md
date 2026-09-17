# Dockerizing an Application

Dockerizing means:

> **Taking an existing application and packaging it into a Docker image so it can run consistently inside a container.**

You've already learned:

```text
Dockerfile
   ↓
docker build
   ↓
Image
   ↓
docker run
   ↓
Container
```

Now we're going to actually do it.

---

# 1. What are we Dockerizing?

Let's use a simple application first.

Since you already know Node.js/Express, imagine:

```text
my-node-app/
│
├── server.js
├── package.json
└── public/
```

The application currently runs directly on Ubuntu:

```bash
npm install
npm start
```

Without Docker:

```text
Ubuntu
 │
 ├── Node.js
 ├── npm packages
 ├── your application
 │
 └── Express server :5050
```

Our goal is:

```text
Ubuntu
 │
 └── Docker
       │
       └── Container
             │
             ├── Node.js
             ├── npm dependencies
             ├── application
             │
             └── Express :5050
```

The important idea:

> **The application doesn't need Node.js installed directly on the host to run inside the container.**

The container provides its user-space runtime.

---

# 2. The Dockerizing workflow

This is the workflow you should memorize:

```text
Existing Application
        │
        ▼
Create Dockerfile
        │
        ▼
docker build
        │
        ▼
Docker Image
        │
        ▼
docker run
        │
        ▼
Containerized Application
```

And if we later use Compose:

```text
Application
    │
    ▼
Dockerfile
    │
    ▼
Image
    │
    ▼
compose.yaml
    │
    ▼
Container
```

---

# 3. The Dockerfile

The most important file when dockerizing an application is:

```text
Dockerfile
```

No extension.

Not:

```text
Dockerfile.txt
```

Just:

```text
Dockerfile
```

It contains **instructions for building the image**.

Think of it as a recipe:

```text
Dockerfile
   │
   ├── What base environment?
   ├── Where should app live?
   ├── What dependencies?
   ├── What files?
   └── What command starts app?
```

---

# 4. Start with the base image

Suppose our app requires Node.js.

We can start with:

```dockerfile
FROM node:24
```

This means:

> Start building my image from an existing image that already contains Node.js 24.

So instead of installing Node manually:

```text
Ubuntu
 ↓
apt
 ↓
Node.js
 ↓
npm
```

we use:

```text
node:24
   ↓
our application
```

Conceptually:

```text
node:24 image
       │
       │ FROM
       ▼
Our image
       │
       ├── Node.js
       ├── npm
       └── our application
```

---

# 5. Set the working directory

Next:

```dockerfile
WORKDIR /app
```

This tells Docker:

> Make `/app` the working directory for subsequent instructions and the application process.

So inside the container:

```text
/
├── bin/
├── etc/
├── usr/
├── ...
└── app/
     └── our application
```

Then commands such as:

```dockerfile
COPY ...
RUN ...
CMD ...
```

can operate relative to `/app`.

---

# 6. Copy dependency files

Suppose our project has:

```text
package.json
package-lock.json
server.js
```

We could do:

```dockerfile
COPY package*.json ./
```

This copies:

```text
package.json
package-lock.json
```

into:

```text
/app/
```

Why copy these separately?

**Docker's build cache.**

Remember our image layers lesson.

We want:

```text
package files
      ↓
npm install
      ↓
application source
```

rather than copying everything immediately.

---

# 7. Install dependencies

Then:

```dockerfile
RUN npm install
```

Docker executes this **during image building**.

That's an important distinction.

```text
docker build
     │
     ├── FROM
     ├── WORKDIR
     ├── COPY
     └── RUN npm install
                    ↑
                 happens NOW
```

The resulting dependencies become part of the image.

When we later run the container:

```bash
docker run ...
```

we don't need to execute `npm install` again.

---

# 8. Copy the application

Now:

```dockerfile
COPY . .
```

This copies the rest of the project into:

```text
/app
```

So our container now conceptually has:

```text
/app
│
├── package.json
├── package-lock.json
├── node_modules/
├── server.js
└── public/
```

---

# 9. Document the application's port

If Express listens on:

```javascript
const PORT = 5050;
```

we can put:

```dockerfile
EXPOSE 5050
```

Remember what we learned earlier:

> `EXPOSE` does **not** publish the port.

It documents:

> "This image expects an application to listen on port 5050."

Actual publishing happens with:

```bash
-p 5050:5050
```

---

# 10. Tell Docker how to start the application

Finally:

```dockerfile
CMD ["node", "server.js"]
```

This defines the default command executed when the container starts.

So:

```text
docker run
    ↓
container starts
    ↓
CMD
    ↓
node server.js
    ↓
Express starts
    ↓
listening :5050
```

---

# 11. Complete Dockerfile

Putting everything together:

```dockerfile
FROM node:24

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5050

CMD ["node", "server.js"]
```

Now let's understand the complete flow:

```text
             Dockerfile
                 │
                 ▼
          FROM node:24
                 │
                 ▼
           WORKDIR /app
                 │
                 ▼
        COPY package*.json
                 │
                 ▼
           RUN npm install
                 │
                 ▼
            COPY . .
                 │
                 ▼
           EXPOSE 5050
                 │
                 ▼
       CMD ["node","server.js"]
                 │
                 ▼
            docker build
                 │
                 ▼
            Docker Image
                 │
                 ▼
             docker run
                 │
                 ▼
             Container
                 │
                 ▼
          Express :5050
```

---

# 12. Building the image

From your application directory:

```bash
docker build -t my-node-app:1.0 .
```

Break it down:

```text
docker build
    │
    └── build an image

-t
    │
    └── give it a name/tag

my-node-app:1.0
    │
    └── image name + tag

.
    │
    └── build context = current directory
```

The `.` is important.

It tells Docker:

> Use the current directory as the build context.

---

# 13. Check your image

```bash
docker images
```

You should find something like:

```text
my-node-app:1.0
```

Now you have:

```text
Dockerfile
     ↓
docker build
     ↓
my-node-app:1.0
```

---

# 14. Run it

Now:

```bash
docker run -d \
  --name my-node-app \
  -p 5050:5050 \
  my-node-app:1.0
```

Architecture:

```text
Browser
   │
   │ localhost:5050
   ▼
Ubuntu Host :5050
   │
   │ Docker port mapping
   ▼
Node Container :5050
   │
   ▼
Express
```

Then:

```bash
docker ps
```

And:

```bash
docker logs my-node-app
```

You should see your Express startup output.

---

# 15. And now `docker exec` becomes useful

Remember what we just learned?

You can inspect your running container:

```bash
docker exec -it my-node-app bash
```

Then:

```bash
pwd
```

should give something like:

```text
/app
```

And:

```bash
node --version
```

shows the Node version **inside the container**.

This is a beautiful demonstration of what Docker actually did:

```text
HOST
Ubuntu
│
│ Docker
▼
CONTAINER
│
├── Node.js
├── npm
├── node_modules
├── application
└── Express
```

---

# 16. One VERY important production concept: `.dockerignore`

Suppose your project contains:

```text
node_modules/
.git/
.env
README.md
```

You generally don't want to blindly copy everything into the image.

Create:

```text
.dockerignore
```

For example:

```text
node_modules
.git
.env
npm-debug.log
```

Then:

```dockerfile
COPY . .
```

won't send those ignored files into the build context.

This is important for:

* security
* build speed
* image size
* avoiding unnecessary files

Especially:

```text
.env
```

because it can contain secrets.

---

# 17. The same idea applies to your FastAPI AI backend

This is where **your actual DevOps path** starts becoming relevant.

Instead of:

```text
Node.js
Express
```

you'll eventually have:

```text
Python
FastAPI
LangChain
LangGraph
your AI application
```

Conceptually:

```text
             Dockerfile
                 │
                 ▼
          Python base image
                 │
                 ▼
        Install dependencies
                 │
                 ▼
          Copy application
                 │
                 ▼
          Start Uvicorn
                 │
                 ▼
          FastAPI Container
```

And then:

```text
                    Docker Compose
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       FastAPI       PostgreSQL       Redis
       :8000           :5432          :6379
          │              │              │
          └──────────────┴──────────────┘
                    Network
```

That is much closer to the type of system you'll eventually deploy.

---

## 🔑 The Dockerizing mental model

Don't memorize the Dockerfile line-by-line yet.

Understand **what each instruction contributes**:

| Instruction | Meaning                               |
| ----------- | ------------------------------------- |
| `FROM`      | Starting environment/image            |
| `WORKDIR`   | Where the application lives           |
| `COPY`      | Put files into the image              |
| `RUN`       | Execute something during image build  |
| `EXPOSE`    | Document intended container port      |
| `CMD`       | Default command when container starts |

And the distinction to permanently remember:

```text
BUILD TIME                         RUN TIME

Dockerfile                         Container
    │                                  ▲
    ▼                                  │
docker build ───────→ Image ──→ docker run
                                      │
                                      ▼
                               CMD executes
```

**`RUN` happens while building the image.**

**`CMD` happens when the container starts.**

That distinction is one of the most important Docker concepts you'll use.
